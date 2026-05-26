# Contexte projet — EduCards Studio Pro

## Vue d'ensemble
Application monofichier HTML (`créateur_de_cartes.html`) qui permet de créer, personnaliser, exporter et imprimer des flashcards éducatives. 100% client-side, persistance via `localStorage` (clé `edu_pro_cards_v2`).

## Stack
- HTML5 + Tailwind CSS (via CDN `cdn.tailwindcss.com`)
- Icônes : `lucide@latest`
- Export image : `html2canvas@1.4.1`
- Polices : Plus Jakarta Sans (Google Fonts)
- Pas de bundler, pas de framework JS, vanilla JS

## Architecture actuelle (créateur_de_cartes.html)
```
<body flex>
 ├─ <aside w-80>  (sidebar gauche fixe, 320px)
 │    ├─ Bouton "Ajouter une carte"
 │    ├─ Recherche
 │    ├─ Exports : JSON / CSV / PNG / Import
 │    ├─ Design global : couleur bordure, épaisseur, radius
 │    ├─ Effacer tout
 │    └─ Imprimer (3/ligne)
 ├─ <main>  (grille de cartes responsive 1/2/3 colonnes)
 └─ <div #editor>  (panneau latéral droit, fixe 400px, slide-in)
```

Format carte :
```js
{ id, question, type ('multiple'|'boolean'), options, answer, image,
  bgColor, styleQ, styleA, styleTitle, styleOptions, imgSize, title }
```

Print A4 : grille 3×3 (60mm × 85mm) avec saut de page tous les 9 cartes.

## Demande utilisateur (cette session)
1. **Responsive** : actuellement la sidebar 320px + editor 400px cassent sur mobile/tablette.
2. **Intuitif + tutoriel** : aucune indication pour un nouvel utilisateur.
3. **Bugs export / impression / PDF** : à identifier et corriger.
4. **Bouton « auto-rescale pro »** : ajuster automatiquement les tailles de police/image pour que tout le contenu rentre dans la carte sans débordement.
5. **Audit complet** : déployer des sous-agents pour identifier bugs, échecs, problèmes UI/UX.

## Branche de travail
`claude/html-responsive-redesign-WxDPN` (basée sur `claude/github-joined-file-9BgMu`).

## Findings agents (audit complet)

### Bugs critiques (à corriger)
- **XSS** : `card.question`, `card.answer`, `card.title`, `card.options` injectés dans `innerHTML` sans échappement → injection JS via import JSON ou saisie.
- **exportPNG** (l.562) : sélecteur `[onclick="openEditor(${card.id})"]` fragile ; clones contiennent les boutons hover ; échec silencieux deck vide ; canvas géant (>16384px) au-delà de ~50 cartes ; ne respecte pas filtre recherche.
- **exportCSV** : pas de BOM UTF-8 (accents Excel cassés), séparateur `,` (FR attend `;`), champ `type`/`options` non échappés.
- **importJSON** : aucune validation, écrase sans confirmation.
- **localStorage quota** : images base64 lourdes saturent ~5MB sans gestion `QuotaExceededError`.
- **id collisions** : `Date.now()` peut produire doublons sur dup rapide.
- **openEditor** ré-injecté à chaque toggle style → perte focus input.
- **moveCard + recherche** : opère sur `cards[]` mais affichage filtré → carte saute hors écran.
- **boolean** : `card.options` jamais purgé, pas de stockage Vrai/Faux comme réponse.
- **globalTitle / border / radius** non persistés.
- **addCard initial** : `window.onload` crée et sauve une carte → état "vierge" impossible.

### Print/PDF
- `@media print` `nth-child(9n+10)` → diverge avec filtre/suppression.
- `max-height: 20mm` sur images en print → coupe la majorité des images.
- `overflow: hidden` masque silencieusement texte long.
- Pas d'export PDF réel (seulement `window.print()`).

### UI/UX & Responsive
- `aside w-80` + `editor w-[400px]` + cartes 320px fixes → cassé < 1024px.
- Pas de hamburger / drawer mobile.
- Pas d'onboarding ; `addCard()` silencieux au 1er chargement.
- `alert/confirm` natifs ; pas de toast ; pas de loader pour exportPNG.
- Boutons icônes sans `aria-label` ; touch targets `p-1` < 44px.
- `text-gray-400` contraste WCAG insuffisant (2.85:1).
- Carte = `<div onclick>` non focusable clavier.
- Chevrons gauche/droite pour ordre vertical → confus.
- Pas d'undo après "Effacer tout".

### Auto-rescale Pro (à implémenter)
Bouton dans Design Global + dans éditeur de carte.
Algorithme : cloner carte off-screen avec `overflow:visible`, mesurer `scrollHeight` vs target 450px, réduire `styleQ.size`/`styleOptions.size` par paliers (min 8px), puis `imgSize` (min 40%), puis padding. Toast final + undo.

## Contraintes
- Garder mono-fichier HTML (déploiement simple)
- Conserver les données existantes en `localStorage` (rétro-compat)
- Conserver la langue française
- Privilégier les CDN existants ; ajouter jsPDF si nécessaire pour vrai PDF
