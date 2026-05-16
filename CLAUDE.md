# coach-landing

Landing page pour l'offre de Romain Wirth : création de landing pages pour coachs de vie, développement personnel et thérapeutes.

## Stack

- **Astro 6** — framework principal (Romain peu familier avec Astro)
- **Tailwind v4** — via `@tailwindcss/vite` (plugin Vite, pas l'intégration Astro)
- **GSAP + ScrollTrigger** — animations (à installer)

## Config particulière

npm nécessite `strict-ssl false` temporairement pour installer des packages (certificat SSL intercepté par l'environnement réseau). Toujours remettre `strict-ssl true` après.

## Charte graphique

Reprise depuis romainwirth.fr — à intégrer par Romain avant de coder les composants.

## Structure prévue

Landing page one-page avec sections dans cet ordre (à affiner) :
- Hero
- Section "Process" — animation scroll horizontal (voir ci-dessous)
- Section suivante qui recouvre le process avec animation

## Animations GSAP

### Section Process (priorité)

- **4 à 5 étapes** (pas plus)
- Effet **pin + scroll horizontal** : la section se fixe à l'écran, le scroll vertical fait défiler les étapes de gauche à droite (scroll bas) ou droite à gauche (scroll haut)
- Durée de scroll : ~150-200% de viewport height
- La **section suivante arrive par-dessus** avec un effet de recouvrement (translateY depuis le bas, ou clip-path — à décider)

### Ordre d'implémentation prévu

1. Structure HTML + cartes placeholder
2. CSS de base
3. GSAP ScrollTrigger scroll horizontal
4. Animation de recouvrement section suivante
