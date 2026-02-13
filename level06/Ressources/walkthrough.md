2. Analyse du binaire
Le programme level06 demande un Login et un Serial.

Le main : Il utilise fgets pour lire le login (limité à 32 octets) et scanf pour le serial.

La fonction auth : Elle contient une protection anti-debug ptrace et calcule un hash basé sur le login.

3. La Faille (Vulnérabilité)
La vulnérabilité réside dans la logique de hachage prévisible :

Cryptographie faible : L'algorithme est déterministe et peut être recréé par un attaquant possédant le binaire.

Confiance excessive : Le programme valide l'accès si l'entrée utilisateur correspond à son calcul interne.

Contournement de protection : La sécurité par ptrace est inutile si l'on calcule le serial à l'extérieur du programme.

4. Résolution pas à pas
Étape 1 : Créer le script dans /tmp
Une fois connecté en SSH, utilise la commande suivante pour créer le script de calcul instantanément :

Bash
cat << 'EOF' > /tmp/solve.py
import sys

def solve(login):
    if len(login) < 6:
        return None
    # Initialisation (index 3 du login)
    res = (ord(login[3]) ^ 0x1337) + 0x5eeded
    # Boucle de hachage
    for char in login:
        if ord(char) < 32: return None
        res += (ord(char) ^ res) % 0x539
    return res

if __name__ == "__main__":
    if len(sys.argv) > 1:
        print(solve(sys.argv[1]))
EOF
Étape 2 : Générer le Serial
Exécute le script avec le login de ton choix (ici marvin42) :

Bash
python /tmp/solve.py marvin42
Le script affichera un nombre (ex: 6234567).

Étape 3 : Exploiter le binaire
Lance le programme et saisis les informations générées :

Bash
./level06
-> Enter Login: marvin42

-> Enter Serial: [Le nombre obtenu à l'étape 2]

Étape 4 : Récupérer le mot de passe
Une fois authentifié, le shell s'ouvre. Tape la commande suivante pour obtenir le flag du niveau suivant:

Bash
cat /home/users/level07/.pass

## 🛡️ Analyse de l'algorithme `auth()`

L'objectif de cette fonction est de transformer une chaîne de caractères (ton login) en un nombre unique (le serial) de manière déterministe.

### 1\. L'Initialisation (Le "Seed")

Avant de commencer la boucle, le programme génère une valeur de départ basée sur le **4ème caractère** de ton login (index `[3]`).

res\=(login\[3\]⊕0x1337)+0x5eeded

-   **XOR (⊕) avec `0x1337`** : Le programme applique un masque binaire sur le caractère. `0x1337` (4919 en décimal) est une constante souvent utilisée dans les challenges de sécurité.
    
-   **Addition de `0x5eeded`** : On ajoute `6 221 293` à ce résultat. Cela place immédiatement la valeur de départ dans une plage de nombres élevée pour paraître complexe.
    

* * *

### 2\. La Boucle d'Accumulation

Le programme itère ensuite sur **chaque caractère** du login (y compris les 6 premiers) pour modifier la valeur de `res`.

resnouveau​\=resactuel​+(login\[i\]⊕resactuel​)(mod0x539)

#### Pourquoi ces opérations ?

-   **`(login[i] ^ res)`** : Cette opération mélange les bits du caractère actuel avec l'état actuel du hash. Cela rend le résultat dépendant de **l'ordre** des lettres. Si tu changes "ABCDEF" en "BACDEF", le serial sera totalement différent.
    
-   **`% 0x539` (Modulo 1337)** : C'est l'étape cruciale. Le modulo limite la valeur ajoutée à chaque tour à une plage comprise entre **0 et 1336**. Cela empêche le nombre de grossir de façon incontrôlée (integer overflow) trop rapidement et force le résultat à rester dans les limites d'un entier 32 bits standard.
    
-   **`res += ...`** : On ajoute ce petit reste au total. Le hash "grandit" petit à petit à chaque caractère du login.

