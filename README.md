# Gestion Documentaire

Application web **monopage** (un seul fichier `index.html`), **100 % locale** : elle ne
nécessite **aucun réseau** et interagit uniquement avec des fichiers sur le disque
(sélecteurs d'ouverture / enregistrement de fichiers).

## Principe

- **Trame** : fichier source Word (.docx), Excel (.xlsx) ou PowerPoint (.pptx) contenant
  des balises de la forme `[Nom de la balise]`.
- **Modèle** : trame dont tout ou partie des balises sont remplacées par une valeur fixe.
- **Document généré** : produit à partir d'un modèle (ou directement d'une trame) en
  complétant les balises restantes ; enregistré sous la forme `prefixe-code-titre.ext`.

*(Documentation complète en cours de rédaction — voir les sections ci-dessous.)*
