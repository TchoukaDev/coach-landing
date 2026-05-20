# coach-landing

Projet Astro contenant **deux landing pages** pour Romain Wirth :

1. **`/`** — Landing page "Création de site web" (offre à 500€ pour coachs/thérapeutes) — terminée
2. **`/offre-accompagnement`** — Landing page "Offre d'accompagnement global" (3 formules : 500€ / 1500€ / 2500€) — squelette en place, contenu à compléter

L'objectif est d'inverser les routes à la fin (accompagnement sur `/`, site web sur une sous-route).

## Stack

- **Astro 6** — framework principal
- **Tailwind v4** — via `@tailwindcss/vite` (plugin Vite, **pas** l'intégration Astro `@astrojs/tailwind`)
- **Vanilla JS** — animations (IntersectionObserver, scroll listeners) — GSAP non installé

## Config particulière

npm nécessite `strict-ssl false` temporairement pour installer des packages (certificat SSL intercepté par l'environnement réseau). Toujours remettre `strict-ssl true` après.

## Structure des composants

```
src/components/
├── Navbar.astro                  — landing site web (/)
├── NavbarAccompagnement.astro    — landing accompagnement (à diverger si besoin)
├── Footer.astro                  — partagé
├── ui/
│   ├── Button.astro              — props: text, variant (primary|secondary), href, target
│   ├── Container.astro           — props: size (sm|md|lg), className
│   └── Section.astro             — props: variant (light|alt|accent|dark), id, className
│                                   + IntersectionObserver fade-in intégré
└── sections/
    ├── landing-site/             — sections de la landing "site web"
    │   ├── Hero.astro
    │   ├── Problems.astro
    │   ├── Solving.astro
    │   ├── ProcessOffer.astro    — animation scroll horizontal + effet overlay (Vanilla JS)
    │   ├── About.astro
    │   ├── Testimonials.astro
    │   ├── FAQ.astro             — accordéon Vanilla JS
    │   └── CTA.astro
    └── accompagnement/           — sections de la landing "accompagnement" (placeholders)
        ├── Hero.astro
        ├── Context.astro
        ├── PainPoints.astro
        ├── Promise.astro
        ├── About.astro
        ├── Method.astro
        ├── Modalities.astro
        ├── Benefits.astro
        ├── Testimonials.astro
        ├── Pricing.astro         — 3 formules (500€ / 1500€ / 2500€)
        ├── Guarantee.astro
        ├── FAQ.astro
        └── CTA.astro
```

## Charte graphique (global.css)

Variables Tailwind v4 définies dans `src/styles/global.css` via `@theme inline` :

- `accent` / `accent-hover` — terracotta (#C4704A / #B8603A)
- `bg-dark` (#0a0a0a), `bg-light` (#FAF7F4), `bg-light-alt` (#F0EAE4)
- `text-on-dark`, `text-on-dark-muted`, `text-on-light`, `text-on-light-muted`
- Fonts : `font-heading` (Playfair Display), `font-body` (DM Sans)

## Patterns à respecter

- Chaque section utilise `<Section variant="...">` + `<Container>` sauf `Hero` (plein écran custom)
- Les sections alternent light / alt / dark selon l'ordre visuel
- L'ID `#hero-cta` doit exister sur le wrapper des boutons du Hero (observé par la Navbar)
- Les imports UI dans `sections/landing-site/` utilisent `../../ui/`, idem pour `sections/accompagnement/`
