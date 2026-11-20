# Gestion Documentaire

**Version 1.2** — application web **monopage** : un seul fichier `index.html`, **aucun accès réseau**.
Ouvrez-le simplement dans un navigateur (double-clic sur le fichier) : tout fonctionne
en local, et les seules entrées/sorties sont des **fichiers sur le disque**.

> La même documentation est embarquée dans l'application : page **Fichier →
> « Exporter la documentation (.md)… »**.

## Hiérarchie : Projet > Types de document / Modèles

- **Projet** (niveau du haut) : sélecteur dans le **bandeau** en haut de l'écran, création
  avec le bouton ＋, gestion (activer / supprimer) dans la page **Fichier**. Les types de
  document et les modèles **appartiennent au projet actif**.
- **Type de document** (nouveau nom de « trame ») : un **nom**, un **trigramme**
  (3 caractères A-Z/0-9, unique dans le projet, ex. `CTR`) et une **trame** (le fichier
  .docx/.xlsx/.pptx contenant les balises `[..]`, détectées automatiquement à l'import,
  même découpées en plusieurs morceaux par Word).
- **Modèle** : appartient à un type de document du projet ; il a son propre **trigramme**,
  qui **remplace l'ancien préfixe** : c'est le trigramme du modèle qui est utilisé dans le
  nom des fichiers générés depuis ce modèle (à défaut, celui du type de document). Tout ou
  partie des balises y sont fixées à une valeur fixe ; les balises laissées vides seront
  demandées à la génération.

## Champs texte enrichis (mise en forme des balises)

Chaque champ de balise (dans un modèle ou à la génération) dispose d'une **barre de mise en
forme** : **Gras, Italique, Souligné, Barré**, couleur et taille (points), qui entoure la
sélection de marqueurs internes : `{b}…{/}`, `{i}…{/}`, `{u}…{/}`, `{s}…{/}`,
`{c=RRGGBB}…{/}`, `{sz=pt}…{/}`. À la génération, ces marqueurs sont convertis en **vrais
éléments formatés** dans le document généré, et la prévisualisation les affiche réellement
mis en forme (gras, couleur, etc.).

**Faisabilité par format (testée) — tous les trois sont supportés :**

| Format | Mise en forme des balises |
|---|---|
| Word (.docx) | ✔ runs `<w:r><w:rPr>…` — même si Word a découpé la balise en plusieurs runs (le run est coupé proprement, le format d'origine est conservé pour le reste) |
| PowerPoint (.pptx) | ✔ runs `<a:r><a:rPr b="1" …><a:solidFill>…` |
| Excel (.xlsx) | ✔ rich text en runs `<r><rPr>…</rPr><t>…</t></r>` (sharedStrings / cellules inline) |

## Interface

- **Bandeau** en haut (style Office, dégradé) : icône, titre + version, **sélecteur de
  projet**, onglets de navigation : 🏠 Nouveau document, 📄 Types de document,
  📋 Modèles, 🗂 Documents & registre, 📁 Fichier. Indicateur d'état de la persistance
  à droite du bandeau.
- **Page principale « Nouveau document »** : **grille de cartes** des modèles et types de
  document du projet actif, **chacune avec une prévisualisation** du contenu de la trame
  (balises surlignées). Clic sur une carte = fenêtre de génération.
- **Fenêtre de génération** : formulaire des balises (avec barres de mise en forme pour les
  champs enrichis) **à côté d'une prévisualisation live** (les balises se remplissent au
  fur et à mesure de la saisie).
- **Bandeau des documents générés** (sous le bandeau principal) : pastilles chrono + titre ;
  clic = page Documents, double-clic = retélécharger.
- **Page « Documents & registre »** (fusion des anciennes pages Documents et Registre) :
  liste des documents générés + **registre Excel consultable** (une table par type de
  document) + export/import manuels du registre.
- **Page « Fichier »** (ex « Paramètres », comme le menu Fichier d'Office) : paramètres,
  gestion des projets, persistance (chemins réseau + « Parcourir… »), export de la
  documentation et du registre.
- **Barre d'état** en bas : projet actif, compteurs, indicateurs de persistance, prochain chrono.

## Nom des fichiers exportés

`aaaammjj_projet_typededocument_chrono - Titre du document.ext`

- `aaaammjj` : date de génération (ex. `20260213`) ;
- `projet` : nom du projet (aplani, ex. `Projet_Alpha`) ;
- `typededocument` : **trigramme** du modèle si génération depuis un modèle, sinon trigramme
  du type de document ;
- `chrono` : **numéro chrono unique** attribué au document (compteur persistant, ex. `000007`) ;
- `Titre du document` : le titre saisi, qui joue le rôle d'une balise `[TITRE]` et est écrit
  dans la **propriété « Titre »** du fichier Office (métadonnée `dc:title`).

## Lire les valeurs des balises d'un document créé

- Dans l'application : page **Documents & registre** → bouton **👁 Valeurs** sur chaque ligne :
  tableau balise → valeur saisie (avec sa mise en forme affichée).
- Dans le **registre Excel** : **une table (feuille) par type de document**
  (`TD <trigramme>`) listant ses documents avec **une colonne par balise** et les valeurs
  utilisées.

## Balises découpées en plages et balise réservée [CHRONO]

- `[XX-n]` / `[XX-n-m]` : seule la balise de base `XX` est détectée ; la valeur de `XX` est
  saisie en entier et seules les positions n (ou n à m, en commençant à 1) sont insérées.
  Exemple : `XX = "ABCDE"` → `[XX-2]` donne `B`, `[XX-2-4]` donne `BCD`. Les plages présentes
  et le **nombre minimum de caractères** attendu sont affichés (fenêtre « Balises »,
  infobulles des cartes/tableaux, aide des formulaires) ; avertissement si la valeur est
  plus courte (caractères manquants remplacés par du vide).
- `[CHRONO]` (réservée, toute casse) : **jamais saisie** — remplie automatiquement avec le
  numéro chrono du document (le même que dans son nom de fichier), y compris découpée
  (`[CHRONO-2]`).

## Mise à jour d'un type de document (rétro-compatible)

La nouvelle version du fichier doit contenir **au minimum les mêmes balises** que
l'ancienne (des balises supplémentaires sont autorisées) ; sinon la mise à jour est
refusée et les balises manquantes sont listées. Les modèles existants restent valides.

## Persistance automatique (dossier des trames + registre Excel)

Pour retrouver projets, types de document, modèles et documents **dès l'ouverture** :

- Dans la page **Fichier**, renseignez le **chemin réseau du dossier des trames** et le
  **chemin réseau du registre Excel** (ex. `\\serveur\partage\trames`), puis associez les
  emplacements réels avec « Parcourir… » (API File System Access : Chrome/Edge ; handles
  conservés en IndexedDB, accès revalidé à chaque ouverture).
- **Dossier des trames** : chaque type de document y est copié (son fichier trame) ; à
  l'ouverture, les fichiers du dossier sont rechargés comme types de document du projet actif.
- **Registre Excel** : réenregistré automatiquement à chaque modification et rechargé à
  l'ouverture (projets, types de document en base64, modèles, documents, chrono, chemins).
- Sans configuration (ou navigateur incompatible) : export/import manuels du registre
  (page Fichier / page Documents & registre).

## Registre Excel (feuilles)

`Paramètres`, `Projets`, `Types de document`, `Modèles`, `Documents`, `Fichiers` (trames en
base64), **plus une feuille `TD <trigramme>` par type de document** : ses documents, une
colonne par balise avec les valeurs utilisées.

## Utilisation pas à pas

1. Créez un **projet** (bouton ＋ du bandeau).
2. Page **Fichier** : chemins réseau + « Parcourir… » pour le dossier des trames et le registre.
3. Page **Types de document** : nom + trigramme + fichier trame → « Créer le type de
   document » (balises détectées automatiquement).
4. Page **Modèles** : choisissez un type de document → « Nouveau modèle… » (nom + trigramme
   + valeurs des balises, éventuellement mises en forme).
5. Page principale : cliquez sur une carte (prévisualisation comprise) → titre + balises
   restantes → « Générer et enregistrer ».
6. Page **Documents & registre** / bandeau : retrouvez les documents, lisez les **valeurs
   par balise** (👁), retéléchargez ; le registre est à jour automatiquement.

## Détails techniques

- Fichiers Office = ZIP de XML : lecteur/écrivain ZIP embarqué (sans bibliothèque externe) ;
  lecture/écriture des parties `word/document.xml`, `ppt/slides/slideN.xml`,
  `xl/sharedStrings.xml`, en-têtes/pieds, notes… Balises recherchées dans le texte concaténé
  de chaque partie (gestion des « runs » Word) ; remplacement reporté dans les bons nœuds XML.
  Les champs enrichis sont appliqués par **édition positionnelle** du XML (découpage du run
  à la position de la balise et insertion de vrais runs formatés).
- Le registre `.xlsx` est généré et lu entièrement en local. Aucune requête réseau n'est émise.

## Prérequis

- Navigateur moderne ; Chrome/Edge recommandés pour la persistance automatique.

## Dépôt

Le projet est versionné avec git (commits incrémentaux par version, tags `v1.0`, `v1.1`,
`v1.2`) : `index.html` (l'application) et `README.md` (cette documentation).
