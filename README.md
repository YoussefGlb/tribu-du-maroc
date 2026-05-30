# Tribus du Maroc — restructuré

## Fichiers

- **`index.html`** — la carte publique. Lit `categories.json` et `tribus_maroc_langues.geojson` au démarrage. Ajouter une section dans `categories.json` fait apparaître automatiquement un onglet ici.
- **`editor.html`** — l'éditeur. Deux modes :
  - **Peintre** : importer un GeoJSON, peindre les tribus avec les catégories existantes, exporter.
  - **Catégories** : CRUD complet sur les sections, items, groupes, couleurs (pickers natifs) et descriptions. Exporter `categories.json`.
- **`categories.json`** — la source unique de vérité pour les sections, items, couleurs, descriptions. Modifiable à la main ou via l'éditeur.
- **`tribus_maroc_langues.geojson`** — les 268 tribus.
- **`demarrer.bat`** — lance un serveur local sur le port 8000 (Windows). Sur Mac/Linux : `python3 -m http.server 8000`.

## Workflow

### Ajouter une catégorie dans une section existante (ex: nouvelle ethnie)
1. Ouvrir `editor.html` (mode **Catégories** en haut)
2. Cliquer la section dans la barre de gauche
3. **+ Ajouter un item** en bas → label → l'éditeur ouvre la fiche
4. Couleur (picker natif), description, parent/groupe si applicable → **Enregistrer**
5. **⬇ Catégories** en haut à droite → remplace `categories.json` dans le dossier
6. Recharger `index.html` (ou cliquer ↺ Reload)

### Ajouter une nouvelle section (ex: Région, Religion, Économie)
1. Mode **Catégories**, **+ Ajouter une section** dans la barre de gauche
2. Label + ID (auto-généré depuis le label) + cocher "hiérarchique" si tu veux des items parents/enfants
3. Ajouter des items dedans
4. Exporter `categories.json` → un nouvel onglet apparaît dans `index.html`
5. Repasse en mode **Peintre**, importer le GeoJSON, peindre les tribus avec les valeurs de la nouvelle section
6. Exporter le GeoJSON

### Modifier une couleur
- Mode **Catégories**, cliquer le carré coloré à gauche de l'item. Picker natif s'ouvre. La modif est immédiate et marque catégories comme modifiées (point doré dans le header). Exporter pour sauvegarder.

### Repeindre une zone qui a changé
1. Mode **Peintre**, onglet section
2. Cliquer un swatch dans la palette pour activer le pinceau
3. Cliquer (ou glisser) sur les tribus à modifier
4. Ctrl+Z pour défaire. Recherche en haut à gauche pour trouver une tribu par nom.
5. **⬇ GeoJSON** → remplace `tribus_maroc_langues.geojson`

## Format de `categories.json`

```jsonc
{
  "version": 1,
  "sections": [
    {
      "id": "ethnicity",                  // devient le nom de propriété sur chaque feature
      "label": "Ethnie & Région",         // affiché dans les onglets
      "description": "...",
      "items": [
        { "id": "chleuh", "label": "Chleuh", "color": "#c4622d", "description": "..." }
      ]
    },
    {
      "id": "origin",
      "label": "Origine",
      "hierarchical": true,               // items peuvent avoir un "parent"
      "subProperty": "sub_origin",        // nom de la propriété sub sur les features (legacy)
      "items": [
        { "id": "masmuda", "label": "Masmuda", "color": "#c4622d", "description": "..." },
        { "id": "awraba",  "label": "Awraba",  "color": "#e07840", "parent": "masmuda", "description": "..." }
      ]
    },
    {
      "id": "language",
      "label": "Langue",
      "groups": [                         // regroupement visuel (Tamazight / Darija)
        { "id": "amazigh", "label": "Tamazight", "color": "#f0c800" },
        { "id": "darija",  "label": "Darija",    "color": "#3060c0" }
      ],
      "overlay": {                        // overlay diagonal (arabisation)
        "enabled": true,
        "label": "Arabisation",
        "property": "arabisation",
        "fromGroup": "amazigh",
        "toGroup": "darija"
      },
      "items": [
        { "id": "tachelhit", "label": "Tachelhit", "color": "#e8b800", "group": "amazigh", "description": "..." }
      ]
    }
  ]
}
```

## Notes

- Les modifications non sauvegardées sont signalées par un point doré dans le header. Une alerte apparaît si tu essaies de fermer l'onglet.
- L'éditeur n'écrit jamais sur le disque automatiquement — toujours via le bouton d'export, puis tu remplaces le fichier à la main.
- Quand tu supprimes une section, les propriétés correspondantes restent dans le GeoJSON (elles deviennent simplement non reconnues). Pas de perte de données silencieuse.
- L'overlay (arabisation) est actuellement codé pour fonctionner entre deux groupes d'une même section. Si tu crées un nouveau cas du même type, ajoute la clé `overlay` à la section.
