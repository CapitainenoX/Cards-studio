# EduCards Studio Pro

> Créateur de flashcards pédagogiques, 100 % web, mono-fichier, sans backend.

Ouvrez `site/créateur_de_cartes.html` dans un navigateur — c'est tout. Aucune installation, aucun build, aucune connexion requise (sauf au premier chargement pour les CDN). Vos cartes sont sauvegardées localement dans le navigateur.

Le dossier `site/` est aussi prêt à être servi tel quel par GitHub Pages, Netlify, Vercel, ou n'importe quel serveur statique — `site/index.html` redirige vers l'app.

## Captures rapides

| Création | Édition | Étude | Impression |
|---|---|---|---|
| Sidebar + grille responsive | Panneau latéral avec styles par champ | Carte plein écran qui se retourne | A4 · 9 cartes/page · prêt à découper |

---

## Fonctionnalités

### Édition
- Cartes 320 × 450 (preview) / 60 × 85 mm (impression A4).
- Deux types : **QCM** (jusqu'à 26 options A–Z) ou **Vrai / Faux**.
- Styles par champ (titre, question, options, réponse) : taille 8–48 px, couleur, gras / italique / souligné.
- Image de fond / illustration (upload PNG/JPG, taille 30–150 %).
- Couleur de fond et bordure par carte ; bordure + radius globaux.
- Réordonnancement (↑/↓), duplication, suppression avec **annulation** (toast 30 s).
- Recherche plein-texte (titre + question + réponse).

### Auto-ajuster (pro)
Mesure le contenu réel de chaque carte sur un clone hors-écran et réduit progressivement les tailles de police puis la taille d'image jusqu'à ce que tout rentre dans 450 px. Bouton dans la sidebar (tout le deck) ou dans l'éditeur (une seule carte). Annulable.

### Mode étude
- Carte plein écran avec animation flip 3D.
- Navigation clavier (← → / Espace / Échap), mélange aléatoire, masquage du verso.
- Indicateur de progression `n / total`.

### Rendu unifié (parité site / aperçu / export)
Chaque carte est dessinée dans un **canvas canonique unique** de 320 × 453 px (ratio 60 × 85 mm), en styles inline. Ce même canvas est réutilisé tel quel par l'aperçu (réduit via `transform: scale`), l'export PNG/PDF, l'impression et la mesure d'auto-ajustement. Résultat : **aucune divergence** de spacing, d'alignement ou de taille entre ce que l'on voit et ce que l'on imprime — fini les titres mal placés ou les chevauchements texte/image dans le PDF.

### Vrai / Faux personnalisable
Les deux réponses et la question d'invite sont entièrement personnalisables (ex : *Oui/Non*, *Correct/Incorrect*, *True/False*). On marque la bonne réponse d'un clic. Pris en charge dans l'aperçu, le mode étude, l'export et l'impression.

### Tutoriel
Lancé automatiquement au premier chargement (6 étapes). **Vrai spotlight** (le reste de la page est assombri, l'élément ciblé reste éclairé), bulle intelligemment positionnée et clampée, comportement cohérent mobile/desktop (ouverture automatique du drawer / de l'éditeur selon l'étape). Re-lançable via « Revoir le tutoriel » ou la touche `T`.

### Exports
| Format | Détails |
|---|---|
| **JSON** | Schéma v3 versionné (`version`, `exportedAt`, `settings`, `cards`). Images réintégrées en base64 → fichier autonome et portable. Réimport compatible v1/v2 (tableau brut). |
| **CSV** | UTF-8 + BOM, séparateur `;` (Excel FR), échappement complet. Libellés V/F préservés. |
| **PNG** | Une image par page A4, rendue offscreen à l'échelle 2× via html2canvas. |
| **PDF** | Multi-pages A4 via jsPDF, JPEG qualité 0.92. |

### Impression
Trois modes via le dialog d'options :
- **Recto + verso (3×3)** — question dessus, réponse renversée en bas, prêt à découper.
- **Recto seul** — quiz : aucune réponse imprimée.
- **Recto/verso séparé (duplex)** — page 1 = questions, page 2 = réponses miroir alignées pour impression double-face.

Action au choix : impression directe (via iframe isolée) ou téléchargement PDF.

### Modèles de démarrage
Trois decks prêts à l'emploi (Sciences, Histoire, Vocabulaire EN) accessibles depuis l'état vide.

### Raccourcis clavier
| Touche | Action |
|---|---|
| `N` | Nouvelle carte |
| `/` | Focus recherche |
| `S` | Mode étude |
| `P` | Imprimer / PDF |
| `A` | Auto-ajuster tout le deck |
| `T` | Lancer le tutoriel |
| `?` | Afficher l'aide |
| `Échap` | Fermer panneau / modal |

---

## Stockage et confidentialité

- **Métadonnées** (texte, styles, libellés) en `localStorage` (clés `edu_pro_cards_v2`, `edu_pro_settings_v1`, `edu_pro_tutorial_done_v1`) — léger, des milliers de cartes tiennent largement.
- **Images** stockées en **Blob dans IndexedDB** (base `educards_db`), pas en base64 dans `localStorage`. On peut ainsi importer des **centaines d'images** sans saturer le quota de ~5 Mo de `localStorage`.
- À l'import, chaque image est **compressée** (redimensionnée à 1280 px max, ré-encodée en **WebP** avec repli JPEG) — les JPEG/JPG/PNG restent parfaitement pris en charge. Object-URLs mis en cache, ramasse-miettes des images orphelines au démarrage.
- **Rétro-compatibilité** : les anciens decks dont les images étaient en base64 dans `localStorage` sont migrés automatiquement vers IndexedDB au premier chargement (ce qui libère le quota). Repli transparent sur `localStorage` si IndexedDB est indisponible (mode privé).
- Aucune donnée n'est envoyée à un serveur. Les CDN (Tailwind, lucide, html2canvas, jsPDF, Google Fonts) sont chargés au démarrage.
- Pour archiver/transférer un deck, **exportez en JSON** (images incluses, fichier autonome).

---

## Stack

| Dépendance | Version | Rôle |
|---|---|---|
| [Tailwind CSS](https://tailwindcss.com/) | CDN | Styles utilitaires |
| [lucide](https://lucide.dev/) | latest | Icônes |
| [html2canvas](https://html2canvas.hertzen.com/) | 1.4.1 | Rendu DOM → canvas pour PNG / PDF |
| [jsPDF](https://github.com/parallax/jsPDF) | 2.5.1 | Génération PDF |
| Plus Jakarta Sans | Google Fonts | Typographie |

Le tout en un seul fichier HTML. Aucun bundler.

---

## Compatibilité

- Chrome / Edge / Firefox / Safari récents (≥ 2 ans).
- Responsive iOS / Android (drawer sidebar + éditeur plein écran sous 768 px).
- Accessibilité : navigation clavier complète, `aria-label` sur les boutons icônes, focus visible WCAG.

---

## Développement

```bash
# Cloner et ouvrir
git clone https://github.com/CapitainenoX/Cards-studio.git
cd Cards-studio
open site/créateur_de_cartes.html   # ou double-clic

# Servir en local
python -m http.server --directory site 8000
# puis http://localhost:8000
```

Pas de build, pas de test runner. Tout vit dans `site/créateur_de_cartes.html`.

### Structure du fichier
```
<head>     # Tailwind config + CSS custom (drawer, modal, tutorial, print)
<body>
  ├── header mobile + sidebar (drawer)
  ├── main (deck + empty state)
  ├── editor (drawer latéral)
  ├── modals (study, print options, shortcuts, confirm)
  └── <script>
        ├── State / load / save
        ├── Toast / modal / busy
        ├── CRUD cartes + render
        ├── Éditeur
        ├── Auto-rescale (mesure offscreen)
        ├── Exports (JSON / CSV / PNG / PDF)
        ├── Print (iframe isolée)
        ├── Tutoriel
        ├── Mode étude
        ├── Raccourcis clavier
        └── Init
```

### Conventions
- État global : `cards[]`, `settings{}` ; métadonnées en `localStorage`, images (Blob) en IndexedDB.
- Rendu de carte : **une seule source de vérité**, `cardCanvasHTML()` (canvas canonique 320×453 en styles inline), partagée par aperçu / PNG / PDF / impression / mesure d'auto-ajustement.
- Pas d'innerHTML avec données utilisateur sans `escapeHtml()` / `escapeAttr()`.
- Toute action destructive passe par `confirmDialog()` (modal custom) + toast d'annulation.

---

## Licence

À définir par le propriétaire du dépôt.
