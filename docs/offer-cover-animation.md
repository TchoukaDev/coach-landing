# Animation — Section Offer (recouvrement au scroll)

> Implémentée dans le script de `Process.astro` — les deux animations partagent
> le même listener scroll et le même wrapper, ce qui évite deux calculs de
> position indépendants.

---

## Principe général

La section Offer est positionnée en `fixed` (hors flux) sur desktop, initialement
hors écran en bas (`translateY(100vh)`). À la fin du scroll du Process, elle monte
et recouvre la section Process qui reste figée sous elle.

---

## Structure

```
<div id="process-wrapper">        ← hauteur étendue (JS) : scroll H + pause + cover
  <section id="process-section">  ← sticky, gère l'animation horizontale
  </section>
</div>

<section id="offer-section"       ← md:fixed inset-0, z-20
  style CSS: translateY(100vh)>   ← hors écran par défaut sur desktop
</section>
```

**Pourquoi `fixed` et pas `sticky` ?**

`sticky` ne devient visible qu'après que le wrapper du Process est entièrement
défilé — le recouvrement se produit trop tard. `fixed` permet à la section Offer
d'être animée *pendant* que le Process est encore à l'écran.

**Pourquoi le script est dans Process.astro et pas Offer.astro ?**

Les deux animations partagent le même `wrapper.offsetTop` et `scrolled`.
Centraliser dans un seul `scroll` listener est plus performant et évite les
dépendances croisées entre composants.

---

## Hauteur du wrapper

```
wrapper.height = 100vh + maxTranslate + coverPause + coverDistance
```

| Segment         | Valeur              | Rôle                                    |
|-----------------|---------------------|-----------------------------------------|
| `100vh`         | viewport height     | Hauteur de la section sticky elle-même  |
| `maxTranslate`  | calculé dynamiquement | Scroll nécessaire pour les cartes      |
| `coverPause`    | `vh * 0.15`         | Pause après la fin du scroll horizontal |
| `coverDistance` | `vh * 0.6`          | Durée de l'animation de recouvrement    |

Sur grand écran (cartes centrées, pas de scroll horizontal) :
```
wrapper.height = 100vh + coverPause + coverDistance
```
`maxTranslate` est absent — seule la phase de recouvrement compte.

---

## Calcul JS

```js
const coverStart = hasHScroll ? maxTranslate + coverPause : coverPause;
const coverProgress = Math.max(0, Math.min(1, (scrolled - coverStart) / coverDistance));
offer.style.transform = `translateY(${(1 - coverProgress) * 100}vh)`;
```

- `coverStart` = le pixel de scroll à partir duquel l'animation commence
- `coverProgress = 0` → `translateY(100vh)` → Offer hors écran
- `coverProgress = 1` → `translateY(0)` → Offer couvre tout le viewport

La pause (`coverPause`) est la zone entre `maxTranslate` et `coverStart` où le
scroll horizontal est terminé mais le cover n'a pas encore commencé — les cartes
restent figées à leur position finale.

---

## Comportement par cas

### Desktop (scroll horizontal actif)

```
scrolled 0 → maxTranslate          : cartes défilent, Offer hors écran
scrolled maxTranslate → +coverPause : pause, tout figé
scrolled +coverPause → +coverDistance : Offer monte de bas en haut
scrolled = fin du wrapper           : Offer plein écran, scroll terminé
```

### Grand écran (toutes les cartes visibles)

```
scrolled 0 → coverPause            : pause, cartes centrées
scrolled coverPause → +coverDistance : Offer monte
scrolled = fin du wrapper           : Offer plein écran
```

### Mobile (< 768px)

Offer en flux normal (`position: static`, `transform: none`), pas d'animation.

---

## Gestion CSS vs inline style

La règle CSS dans Offer.astro empêche le flash au chargement :

```css
@media (min-width: 768px) {
  #offer-section { transform: translateY(100vh); }
}
```

Le JS écrase ensuite ce style via `offer.style.transform`. Les cas particuliers :

| Cas                    | Ce que fait le JS                                  |
|------------------------|----------------------------------------------------|
| Desktop (animation)    | `offer.style.position = ""` → `md:fixed` s'applique |
| Grand écran (no hscroll) | `offer.style.position = "static"` → annule `md:fixed` |
| Mobile                 | `offer.style.transform = ""; offer.style.position = ""` |

Sur grand écran, `position: static` est nécessaire pour annuler `md:fixed` de
Tailwind — `transform: ""` seul ne suffirait pas : la règle CSS
`translateY(100vh)` reprendrait le dessus et l'Offer resterait hors écran.

---

## Récapitulatif visuel (desktop)

```
Phase 1 — scroll horizontal :
┌──────────── viewport ────────────┐
│  [01 Appel]  [02 Cadrage]  [03  │   Offer : invisible (sous le fold)
└──────────────────────────────────┘

Phase 2 — pause :
┌──────────── viewport ────────────┐
│  [03 Création]  [04 Livraison]  │   Offer : encore hors écran
└──────────────────────────────────┘

Phase 3 — recouvrement :
┌──────────── viewport ────────────┐
│  [03 Création]  [04 Livraison]  │
│▓▓▓▓▓▓▓▓▓▓ OFFER (monte) ▓▓▓▓▓▓▓│
└──────────────────────────────────┘

Fin :
┌──────────── viewport ────────────┐
│                                  │
│     500€   ✓ Design sur-mesure   │
│            ✓ Copywriting ...     │
└──────────────────────────────────┘
```
