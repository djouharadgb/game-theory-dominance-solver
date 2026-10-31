# game-theory-dominance-solver

Élimination itérée des stratégies dominées (dominance **stricte** et **faible**) pour un jeu fini sous forme normale,
avec un nombre quelconque de joueurs et de stratégies par joueur. Seules les stratégies pures sont considérées.

TP de théorie des jeux, Master 2 – FI/Département IA&SD.

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `dominance_solver.ipynb` | Le programme complet (code + explications + exécution des jeux de test) |
| `exemple1.json` | Exemple de fichier de jeu (l'exemple 1 du TP), créé et relu par le notebook |
| `README.md` | Ce document |

## Comment lancer le programme

1. Installer Jupyter si besoin :
   ```
   pip install notebook
   ```
2. Se placer dans le dossier du projet et lancer :
   ```
   jupyter notebook dominance_solver.ipynb
   ```
   (ou ouvrir le `.ipynb` dans VS Code, ou dans Google Colab).
3. Menu **Run → Run All Cells**. Toutes les cellules s'exécutent sans saisie, et affichent les traces des jeux de test.

Aucune bibliothèque externe n'est nécessaire : seulement `itertools`, `random` et `json` (modules standard de Python).

Pour tester **avec ses propres données** :

```python
noms, gains = saisie_clavier()                       # saisie au clavier
mode = choisir_mode()                                # "strict" ou "faible"
res = elimination_iteree(gains, noms, mode=mode)
```

(Ces lignes sont en commentaire dans le notebook, car elles demandent des saisies : enlever les `#`.)

## Comment un jeu est représenté

* `noms` : liste de listes. `noms[0]` = stratégies du joueur 1, `noms[1]` = celles du joueur 2, etc.
* `gains` : dictionnaire `{profil : gains}`, où le profil est un tuple de stratégies (une par joueur)
  et les gains un tuple de nombres (un par joueur).

```python
noms  = [["H", "B"], ["G", "D"]]
gains = {("H", "G"): (3, 1), ("H", "D"): (2, 3),
         ("B", "G"): (2, 2), ("B", "D"): (0, 1)}
```

Un joueur peut avoir un nombre de stratégies différent des autres.

## Explication du code

### R1 – Saisie du jeu

| Fonction | Rôle |
|---|---|
| `saisie_clavier()` | Demande le nombre de joueurs, le nombre de stratégies de chacun, puis les gains de chaque profil. Les stratégies sont nommées A1, A2, ... (joueur 1), B1, B2, ... (joueur 2), etc. |
| `lire_fichier(chemin)` / `sauvegarder_fichier(chemin, noms, gains)` | Lecture et écriture d'un jeu au format JSON. |
| `jeu_aleatoire(tailles)` | Jeu aléatoire. `tailles = [3, 2, 2]` donne 3 joueurs avec 3, 2 et 2 stratégies. Gains entiers de 0 à 10. |
| `jeu_depuis_matrice(...)` | Aide pour écrire à la main les jeux à 2 joueurs des exemples. |

Format du fichier JSON : la liste des stratégies de chaque joueur, puis une ligne par profil (les stratégies, suivies des gains).
Voir `exemple1.json`.

### R2 – Affichage

* **2 joueurs** : une matrice (joueur 1 en lignes, joueur 2 en colonnes), chaque case contient le vecteur de gains.
* **3 joueurs ou plus** : joueurs 1 et 2 en lignes et colonnes, et **une matrice par combinaison** de stratégies des joueurs 3 à n,
  étiquetée (par exemple `--- J3=C ---`).

Seules les stratégies encore actives sont affichées, ce qui montre le jeu après chaque suppression.

### R3 et R4 – Test de dominance : `domine(gains, actives, i, b, a, mode)`

Réponse à : « la stratégie `b` du joueur `i` domine-t-elle `a` ? ».

On parcourt toutes les combinaisons de stratégies encore actives des **autres** joueurs, et pour chacune on compare le gain
du joueur `i` avec `b` et avec `a`. On retient :

* `b` strictement meilleure partout ;
* `b` meilleure ou égale partout ;
* `b` strictement meilleure au moins une fois.

Ensuite :

* **mode `strict`** : il faut « strictement meilleure partout » ;
* **mode `faible`** : il faut « meilleure ou égale partout » **et** « strictement meilleure au moins une fois ».

Deux stratégies avec des gains identiques ne se dominent pas (ni en strict, ni en faible), comme demandé par l'énoncé.

### R5 et R6 – Élimination itérée : `elimination_iteree(gains, noms, mode, ordre_joueurs, trace)`

1. On part de toutes les stratégies.
2. `trouver_elimination` cherche **une** stratégie dominée et la stratégie qui la domine.
3. On la supprime et on affiche : le joueur, la stratégie supprimée, la stratégie dominante, le jeu obtenu.
4. On recommence jusqu'à ce qu'il n'y ait plus rien à supprimer.
5. `afficher_resultat` affiche le jeu réduit et indique s'il reste un seul profil (avec ses gains) ou plusieurs.

Choix de conception :

* **Une suppression à la fois**, pour avoir une trace claire (une ligne par étape).
* **Ordre de parcours des joueurs** : par défaut J1, J2, ..., Jn. On peut le changer avec `ordre_joueurs`, par exemple
  `ordre_joueurs=[1, 0]` pour J2 puis J1. Le programme **affiche toujours l'ordre suivi**.
* Si plusieurs stratégies en dominent une autre, on indique la **première trouvée** (dans l'ordre des stratégies).
* En dominance **stricte**, l'ordre ne change pas le jeu réduit final. En dominance **faible**, il peut le changer (voir l'exemple 3).

## Résultats sur les jeux de test

Tous les tests de la section 5 du TP donnent le résultat attendu. Le notebook les vérifie automatiquement dans sa dernière partie.

| Jeu | Mode | Jeu réduit obtenu | Attendu | Test |
|---|---|---|---|---|
| Exemple 1 | strict | (H, C), gains (2, 3) | (H, C), gains (2, 3) | OK |
| Exemple 2 (Pierre-Feuille-Ciseaux) | strict | matrice 3×3 inchangée | aucune élimination | OK |
| Exemple 2 (Pierre-Feuille-Ciseaux) | faible | matrice 3×3 inchangée | aucune élimination | OK |
| Exemple 3 | strict | `{H, B} × {G, M}` (D supprimée) | matrice 2×2 (G, M) | OK |
| Exemple 3, ordre J1 puis J2 | faible | `H × {G, M}` | `H × {G, M}` | OK |
| Exemple 3, ordre J2 puis J1 | faible | `{H, B} × G` | `{H, B} × G` | OK |
| Exemple 4 (3 joueurs) | strict | (N, N, N), gains (0, 0, 0) | (N, N, N), gains (0, 0, 0) | OK |
| Exemple 5 (Série 1, exercice 4) | strict | (A3, B3), gains (5, 3) | (A3, B3), gains (5, 3) | OK |
| Exemple 5 (Série 1, exercice 4) | faible | (A3, B3), gains (5, 3) | (A3, B3), gains (5, 3) | OK |

### Exemple 1 (strict) : trace obtenue

```
Étape 1 : le joueur 2 supprime D, strictement dominée par G
Étape 2 : le joueur 1 supprime M, strictement dominée par H
Étape 3 : le joueur 1 supprime B, strictement dominée par H
Étape 4 : le joueur 2 supprime G, strictement dominée par C
Il reste un seul profil : ('H', 'C') avec les gains (2, 3)
```

Remarque : l'énoncé dit que D est dominée par C « et aussi par G ». Le programme affiche la première dominante trouvée, ici G.
Le résultat final est le même.

### Exemple 2 (Pierre-Feuille-Ciseaux) : aucune élimination

```
Aucune élimination n'est possible : le jeu reste inchangé.
Il reste plusieurs profils : le jeu n'est pas réduit à un seul profil.
```

### Exemple 3 : strict, puis faible avec deux ordres

* **Strict** : seule `D` est supprimée (dominée par `G`). Il reste `{H, B} × {G, M}`.
* **Faible, ordre J1 puis J2** : J1 supprime `B` (dominée faiblement par `H`), puis J2 supprime `D`. Résultat : `H × {G, M}`.
* **Faible, ordre J2 puis J1** : J2 supprime `M` (dominée faiblement par `G`), puis `D`. Résultat : `{H, B} × G`.

Cela confirme que la dominance faible peut donner des jeux réduits différents selon l'ordre. C'est pour cela que le programme affiche l'ordre suivi.

### Exemple 4 (3 joueurs, strict) : trace obtenue

Le jeu de départ est affiché avec deux matrices (`J3=C` et `J3=N`).

```
Étape 1 : le joueur 1 supprime C, strictement dominée par N
Étape 2 : le joueur 2 supprime C, strictement dominée par N
Étape 3 : le joueur 3 supprime C, strictement dominée par N
Il reste un seul profil : ('N', 'N', 'N') avec les gains (0, 0, 0)
```

### Exemple 5 (strict et faible) : trace obtenue

Les deux modes donnent la même suite d'étapes, identique à celle de l'énoncé (le programme écrit « strictement dominée » ou « faiblement dominée » selon le mode) :

```
Étape 1 : le joueur 2 supprime B2, dominée par B3
Étape 2 : le joueur 1 supprime A2, dominée par A1
Étape 3 : le joueur 2 supprime B1, dominée par B3
Étape 4 : le joueur 1 supprime A1, dominée par A3
Il reste un seul profil : ('A3', 'B3') avec les gains (5, 3)
```

## Ce qui fonctionne

* Saisie par clavier, par fichier JSON et par génération aléatoire, pour n joueurs et des nombres de stratégies différents.
* Affichage d'une matrice (2 joueurs) ou de plusieurs matrices étiquetées (3 joueurs ou plus).
* Dominance stricte et faible, au choix de l'utilisateur.
* Élimination itérée avec trace complète et résultat final.
* Les 5 jeux de test de l'énoncé.
* Des jeux aléatoires à 2 et 3 joueurs ont aussi été exécutés dans le notebook, sans erreur.

## Ce qui ne fonctionne pas ou reste limité

* **Pas d'interface graphique** : tout se passe dans le notebook, avec une sortie texte.
* **Dominance faible et ordre** : le programme suit un seul ordre par exécution (celui donné par `ordre_joueurs`).
  Il n'explore pas automatiquement tous les ordres possibles. Pour comparer, il faut relancer avec un autre ordre.
* **Dominance par des stratégies pures uniquement** : une stratégie n'est jamais déclarée dominée par un mélange de stratégies (hors du cadre du TP).
* **Saisie au clavier** : les stratégies sont nommées automatiquement (A1, A2, B1, ...), on ne peut pas choisir les noms
  (le format fichier JSON, lui, accepte n'importe quels noms).
* **Taille des jeux** : le nombre de profils croît très vite avec le nombre de joueurs (produit des tailles). Cela convient pour un TP, pas pour de très gros jeux.
* **Mise en page** : les colonnes sont alignées pour des noms de stratégies courts (moins de 13 caractères).
