# Gestion Documentaire

**Version 1.0** — application web **monopage** : un seul fichier `index.html`, **aucun accès réseau**.
Ouvrez-le simplement dans un navigateur (double-clic sur le fichier) : tout fonctionne
en local, et les seules entrées/sorties sont des **fichiers sur le disque**.

> La même documentation est embarquée dans l'application : page **Paramètres →
> « Exporter la documentation (.md)… »**.

## Interface

- **Page principale « Nouveau document »** (au centre de l'écran) : une **grille de
  cartes** présentant les **modèles** et les **trames**. Cliquez sur une carte pour
  générer un document (depuis le modèle, ou en complétant les balises restantes de la
  trame).
- **Bandeau** sous le menu du haut : les **documents générés** (code + titre) ;
  clic = voir dans la page Documents, double-clic = retélécharger le fichier.
- **Pages secondaires** (menu du haut) :
  - `Trames` : import des fichiers de trames, liste, mise à jour rétro-compatible,
    création de modèles, génération ;
  - `Modèles` : création / modification / suppression, génération ;
  - `Documents` : liste complète des documents générés (code unique, nom
    `prefixe-code-titre.ext`, chrono), retéléchargement ;
  - `Registre` : consultation du tableau Excel complet ;
  - `Paramètres` : chemins du dossier des trames et du registre Excel, export/import
    manuels, documentation.
- **Barre d'état** en bas : compteurs, préfixe des documents, prochain code et chrono.

## Persistance automatique (dossier des trames + registre Excel)

Pour retrouver trames, modèles et documents **dès l'ouverture de l'application** :

- Dans **Paramètres**, renseignez le **chemin réseau du dossier des trames** et le
  **chemin réseau du registre Excel** (ex. `\\serveur\partage\trames`), puis associez
  les emplacements réels avec « Parcourir… » (API File System Access : Chrome/Edge ;
  l'accès est redemandé à chaque ouverture). Les emplacements choisis sont conservés
  localement (IndexedDB).
- **Dossier des trames** : chaque trame importée ou mise à jour y est copiée ; à
  l'ouverture, les trames du dossier sont rechargées automatiquement.
- **Registre Excel** : à chaque modification (trame, modèle, document, paramètre), le
  registre `.xlsx` est réenregistré automatiquement dans ce fichier ; à l'ouverture, il
  est rechargé (trames en base64, modèles, documents, préfixe, compteurs de codes et
  de chrono, chemins).
- Sans configuration (ou navigateur incompatible), l'export/import manuel du registre
  reste disponible (Paramètres / page Registre).

## Concepts

- **Trame** : fichier source Word (.docx), Excel (.xlsx) ou PowerPoint (.pptx) contenant
  des **balises** de la forme `[Nom de la balise]` dans son texte. À l'import,
  l'application lit le fichier et **détecte automatiquement les balises** (même si
  Word les a découpées en plusieurs morceaux internes).
- **Mise à jour d'une trame** : la nouvelle version doit être **rétro-compatible**
  (au minimum les mêmes balises) ; sinon elle est refusée et les balises manquantes
  sont listées. Les modèles existants restent valides.
- **Modèle** : trame dont **tout ou partie des balises** ont été remplacées par une valeur
  fixe. Les balises laissées vides seront demandées à la génération.
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
  - `prefixe` est un **paramètre de l'application** (barre d'état, en bas à droite, `GD` par défaut) ;
  - `code` est le code unique ci-dessus ;
  - `titre` est le **titre du document**, qui joue le rôle d'une balise : s'il existe une
    balise `[TITRE]` dans la trame, elle est remplie avec cette valeur, et dans tous les cas
    le titre est écrit dans la **propriété « Titre » du document** (métadonnée du fichier
    Office) et dans le nom du fichier.

## Registre Excel

L'application conserve **en mémoire** les **fichiers de trames** et la **liste des trames,
des modèles et des documents générés**, sous la forme d'un **tableau Excel** (`.xlsx`) :
consultable dans la page `Registre`, enregistré automatiquement dans le fichier configuré
(voir Paramètres), et exportable / réimportable manuellement.

Le classeur contient 5 feuilles : `Paramètres`, `Trames`, `Modèles`, `Documents`,
`Fichiers` (contenu des trames en base64).

## Utilisation pas à pas

1. Page **Paramètres** : renseignez les chemins réseau et associez le dossier des trames
   et le registre Excel (« Parcourir… »). Réglez le **préfixe** dans la barre d'état.
2. Page **Trames** → « Importer des trames… » : sélectionnez vos fichiers `.docx`/`.xlsx`/`.pptx`
   à balises `[..]`. Ils sont copiés dans le dossier des trames.
3. Créez des **modèles** (pages Trames ou Modèles) en fixant tout ou partie des balises ;
   ou mettez à jour une trame (version rétro-compatible exigée).
4. Page principale **« Nouveau document »** : cliquez sur une carte de modèle ou de trame →
   saisissez le **titre**, complétez les **balises restantes** → « Générer et enregistrer ».
5. Retrouvez les documents dans le **bandeau** (haut) ou la page **Documents** ; le
   registre est à jour automatiquement.

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

- Un navigateur moderne ; Chrome/Edge recommandés pour la persistance automatique
  (File System Access + IndexedDB). Sans cela, export/import manuels du registre.

## Dépôt

Le projet est versionné avec git (commits incrémentaux par version propre) :
`index.html` (l'application) et `README.md` (cette documentation).
