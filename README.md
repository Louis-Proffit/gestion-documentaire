# Gestion Documentaire

Application web **monopage** : un seul fichier `index.html`, **aucun accès réseau**.
Ouvrez-le simplement dans un navigateur (double-clic sur le fichier) : tout fonctionne
en local, et les seules entrées/sorties sont des **fichiers sur le disque**
(sélecteurs d'ouverture, enregistrements proposés via le sélecteur de fichiers du
navigateur, ou téléchargement selon le navigateur).

> La même documentation est embarquée dans l'application : onglet **Aide →
> Documentation (markdown)** (ou onglet **Fichier → Exporter la documentation**) pour
> l'exporter en `.md`.

## Interface (façon suite Office)

- **Barre de titre** avec icône de l'application et zone de recherche
  « Dites-nous ce que vous voulez faire » : tapez quelques mots pour lancer directement
  n'importe quelle commande du ruban.
- **Ruban** à onglets : `Fichier` (vue backstage : informations, enregistrer / ouvrir le
  registre Excel, exporter la documentation), `Accueil`, `Trames`, `Modèles`, `Documents`,
  `Registre`, `Aide`. Chaque onglet regroupe ses commandes par **groupes**, avec des
  **menus déroulants** (▾) pour les variantes (ex. Ouvrir → registre Excel ou fichiers de
  trames).
- Les **tableaux** se sélectionnent au clic (ligne bleue) : les commandes du ruban agissent
  sur la sélection et sont désactivées tant qu'aucune ligne n'est sélectionnée.
  **Double-clic** sur une ligne = action principale (générer un document, retélécharger).
- **Barre d'état** en bas : compteurs (trames / modèles / documents), champ **préfixe**
  des documents et prochain code unique.

## Concepts

- **Trame** : fichier source Word (.docx), Excel (.xlsx) ou PowerPoint (.pptx) contenant
  des **balises** de la forme `[Nom de la balise]` dans son texte. À l'import,
  l'application lit le fichier et **détecte automatiquement les balises** (même si
  Word les a découpées en plusieurs morceaux internes).
- **Mise à jour d'une trame** : on peut remplacer une trame par une nouvelle version du
  fichier. La mise à jour n'est acceptée que si elle est **rétro-compatible**, c'est-à-dire
  si la nouvelle version contient **au minimum les mêmes balises** que l'ancienne
  (des balises supplémentaires sont autorisées). Sinon, la mise à jour est refusée et
  les balises manquantes sont listées. Les modèles existants restent donc valides.
- **Modèle** : trame dont **tout ou partie des balises** ont été remplacées par une valeur
  fixe. Les balises laissées vides dans le modèle seront demandées au moment de la génération.
- **Document généré** : obtenu à partir d'un modèle (valeurs pré-remplies) ou directement
  d'une trame, en complétant les balises restantes. L'application propose alors
  **d'enregistrer le document** à l'emplacement de votre choix.

## Balises découpées en plages de caractères

Si une trame contient une balise de la forme `[XX-n]` ou `[XX-n-m]`, l'application ne
détecte que la **balise de base `XX`** : la valeur saisie pour `XX` (dans le modèle ou à
la génération) est fournie en entier, et seules les positions demandées sont insérées :

- `[XX-n]` → remplacé par le **n-ième caractère** de la valeur (en commençant à 1) ;
- `[XX-n-m]` → remplacé par les **caractères n à m** de la valeur (toujours en commençant à 1).

Exemple : valeur `XX = "ABCDE"` → `[XX-2]` donne `B`, `[XX-2-4]` donne `BCD`.
L'application **informe des plages présentes dans la trame** (fenêtre « Balises », infobulle
des balises dans le tableau, et aide dans les formulaires de modèle/génération), notamment
le **nombre minimum de caractères** attendu pour chaque balise découpée. Si la valeur saisie
est plus courte, un avertissement est affiché (les caractères manquants sont remplacés par
du vide).

## Balise réservée [CHRONO]

La balise `chrono` (peu importe la casse) est **réservée** : sa valeur est le **numéro chrono**
généré automatiquement par l'application (compteur persistant, ex. `000007`, enregistré dans
le registre). Il est **interdit de saisir sa valeur** dans un modèle ou un document à générer :
le champ n'est jamais proposé, et à chaque génération d'un document, un nouveau numéro chrono
est attribué et toutes les occurrences `[CHRONO]` (y compris découpées, ex. `[CHRONO-2]`) sont
remplies avec cette valeur. Le numéro chrono utilisé figure dans la liste des documents générés
(colonne « Chrono ») et dans le registre Excel.

## Nommage et identité des documents générés

- Chaque document reçoit un **code unique** de la forme `AAAA-NNNN` (année + compteur
  séquentiel persistant, ex. `2026-0007`).
- Le fichier généré est nommé **`prefixe-code-titre.ext`**, par ex.
  `GD-2026-0007-Contrat ACME.docx` :
  - `prefixe` est un **paramètre de l'application** (champ en haut à droite, `GD` par défaut) ;
  - `code` est le code unique ci-dessus ;
  - `titre` est le **titre du document**, qui joue le rôle d'une balise : s'il existe une
    balise `[TITRE]` dans la trame, elle est remplie avec cette valeur, et dans tous les cas
    le titre est écrit dans la **propriété « Titre » du document** (métadonnée du fichier
    Office) et dans le nom du fichier.

## Registre Excel

L'application conserve **en mémoire** :

- les **fichiers de trames** eux-mêmes, et
- la **liste des trames, des modèles et des documents générés**,

le tout sous la forme d'un **tableau Excel** (`.xlsx`) que vous pouvez :

- **consulter dans l'application** (onglet « Registre Excel ») ;
- **exporter** (bouton « Exporter le registre Excel ») pour le conserver sur le disque ;
- **réimporter** (bouton « Importer un registre Excel ») pour restaurer toute la session
  (trames, modèles, documents, préfixe, compteur de codes).

Le classeur contient 5 feuilles : `Paramètres`, `Trames`, `Modèles`, `Documents`,
`Fichiers` (contenu des trames en base64).

## Utilisation pas à pas

1. Renseignez le **préfixe** des documents dans la barre d'état (en bas à droite).
2. Onglet **Trames** → sélectionnez un ou plusieurs fichiers `.docx`/`.xlsx`/`.pptx`
   contenant des balises `[..]` → « Importer ces trames ». Les balises détectées
   apparaissent dans le tableau.
3. Sur une trame : « Créer un modèle » pour fixer tout ou partie des balises,
   ou « Mettre à jour » pour charger une version rétro-compatible du fichier.
4. Onglet **Modèles** (ou directement depuis une trame) → « Générer un document » :
   saisissez le **titre**, complétez les **balises restantes**, puis validez.
5. Choisissez où **enregistrer le document généré** (nommé `prefixe-code-titre.ext`).
6. Onglet **Documents générés** : retrouvez la liste avec les codes uniques et
   re-téléchargez un document si besoin.
7. Onglet **Registre Excel** : consultez le tableau ; bouton « Exporter le registre
   Excel » pour le sauvegarder sur le disque.

## Détails techniques

- Un fichier Office (docx/xlsx/pptx) est une archive ZIP de XML : l'application embarque
  son propre **lecteur/écrivain ZIP** (sans aucune bibliothèque externe) et lit/écrit
  directement les parties XML concernées (`word/document.xml`, `ppt/slides/slideN.xml`,
  `xl/sharedStrings.xml`, en-têtes/pieds de page, notes, etc.).
- Les balises sont recherchées dans le **texte concaténé** de chaque partie, ce qui gère
  les balises découpées en plusieurs « runs » par Word ; le remplacement est reporté dans
  les bons nœuds XML en conservant la mise en forme du document.
- Le titre est écrit dans `docProps/core.xml` (propriété `dc:title`).
- Le registre Excel est un vrai fichier `.xlsx` généré et lu entièrement en local.
- Aucune donnée ne quitte le navigateur : aucune requête réseau n'est émise.

## Prérequis

- Un navigateur moderne (Chrome, Edge, Firefox, Safari récents).
- L'enregistrement « à l'emplacement choisi » utilise l'API File System Access quand elle
  est disponible ; sinon un téléchargement classique est proposé.

## Dépôt

Le projet est versionné avec git (commits incrémentaux par version propre) :
`index.html` (l'application) et `README.md` (cette documentation).
