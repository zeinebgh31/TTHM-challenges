# PWN101 challenge

## basic check

checksec --file=pwn101-1644307211706.pwn101
RLEO : full releo
STACK CANARY : No canary found
NX : NX enabled
PIE : PIE enabled

> pas de protection canary donc on peut executer une attaque buffer overflow

## executer le fichier

chmod + pwn102-1644307211706.pwn102
./pwn102-1644307211706.pwn102
       ┌┬┐┬─┐┬ ┬┬ ┬┌─┐┌─┐┬┌─┌┬┐┌─┐
        │ ├┬┘└┬┘├─┤├─┤│  ├┴┐│││├┤
        ┴ ┴└─ ┴ ┴ ┴┴ ┴└─┘┴ ┴┴ ┴└─┘
                 pwn 102

I need badf00d to fee1dead
Am I right? no
I'm feeling dead, coz you said I need bad food :(

## resumé du main

Setup et affichage de bannière

Lit une chaîne utilisateur avec scanf() dans une variable locale

Compare deux variables locale initalisé à 0xbadf00d et 0xlle2153 aux valeurs 0xc0ff33 et 0xc0d3

Si égal → affiche message, exécute une commande shell via system()

Sinon →affiche message erreur et quitte

## approche de l'exploit

Dans le code on remarque quand a la fonction scanf qui est une fonction vulnerable . Quand on interprete les arrguments de scanf on remarque qu'il lit les input dans un format specefique , pour savoir exactement c'est quoi le format on cherche dans Ghidra l'adresse a quoi correspond exactement . Dans notre cas elle correspont a un string
**Note**
la fonction scanf lit la chaine jusqu'elle arrive au null byte elle s'arréte (dans le null byte est un bad char dans ce cas)
Donc on a un buffer de taille 104 octets on va le remplir pour reecrire les 2 variables que se situent apres avec les valeurs 0xc0ff33 et 0xc0d3
