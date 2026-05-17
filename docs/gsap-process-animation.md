# Animation GSAP — Section Process

## Ce que fait l'animation

La section Process présente 4 étapes en scroll horizontal sur desktop.
Quand l'utilisateur scrolle vers le bas, les cartes défilent de gauche à droite.
La section reste fixe à l'écran pendant toute la durée du scroll.

Sur mobile (< 768px) : les cartes sont empilées verticalement, aucune animation.

---

## Structure HTML impliquée

```
<div class="overflow-hidden">          ← clips le débordement visuel des cartes
  <section id="process-section">       ← élément pîné par GSAP
    <ol id="process-track">            ← élément déplacé horizontalement
      <li> ... </li>                   ← cartes (md:min-w-[32vw])
      <li aria-hidden> </li>           ← spacer droit (md:min-w-[8vw])
    </ol>
  </section>
</div>
```

**Pourquoi le wrapper overflow-hidden est séparé de la section pinée ?**
GSAP insère un "spacer" dans le DOM quand il pine un élément (pour compenser la place qu'il prend dans le flux). Si overflow-hidden est sur la section pinée elle-même, ça interfère avec ce mécanisme et l'animation se casse.

**Pourquoi un spacer `<li>` à la fin ?**
`scrollWidth` du track inclut le contenu mais pas toujours le padding CSS droit dans un flex overflow. Le spacer est une solution fiable pour garantir un espace après la dernière carte.

---

## Le script GSAP

```js
gsap.registerPlugin(ScrollTrigger);
```
ScrollTrigger est un plugin séparé — il doit être enregistré avant d'être utilisé.

---

```js
gsap.matchMedia().add("(min-width: 768px)", () => { ... });
```
L'animation ne s'applique qu'en desktop (≥ 768px).
Quand on passe en dessous, GSAP nettoie automatiquement le pin et les transforms.

---

```js
gsap.to(track, {
  x: () => -(track.scrollWidth - window.innerWidth),
  ...
});
```
Déplace le track vers la gauche.

- `track.scrollWidth` = largeur totale du track (toutes les cartes + spacer)
- `window.innerWidth` = largeur visible du viewport
- La différence = distance à parcourir pour tout voir
- Le signe `-` = vers la gauche

**Pourquoi une fonction `() =>`  et pas une valeur directe ?**
Avec `invalidateOnRefresh: true`, GSAP recalcule les valeurs quand la fenêtre est redimensionnée. Pour que ça fonctionne, les valeurs doivent être des fonctions (appelées au moment du recalcul), pas des nombres figés au chargement.

---

```js
scrollTrigger: {
  trigger: section,   // l'élément qui déclenche l'animation
  start: "top top",   // quand le haut de la section atteint le haut du viewport
  end: () => `+=${track.scrollWidth - window.innerWidth}`, // durée en pixels de scroll
  pin: true,          // fixe la section pendant l'animation
  scrub: 1,           // lie le mouvement au scroll (1 = léger délai en secondes)
  invalidateOnRefresh: true, // recalcule au resize
}
```

### Détail de `start` et `end`

`start: "top top"` — syntaxe GSAP : `"[repère sur l'élément] [repère sur le viewport]"`

| Valeur | Signification |
|--------|--------------|
| `"top top"` | haut de la section → haut de l'écran |
| `"top center"` | haut de la section → milieu de l'écran |
| `"center top"` | milieu de la section → haut de l'écran |

`end: () => `+=${distance}`` — le `+=` signifie "X pixels de scroll supplémentaires après le start".
Ici, on scrolle autant de pixels qu'il y a de contenu à révéler — le pin dure exactement le temps de traverser toutes les cartes.

### Détail de `scrub`

| Valeur | Effet |
|--------|-------|
| `true` | suivi immédiat du scroll |
| `1` | léger délai de 1 seconde (effet fluide) |
| `2` | délai plus marqué |

---

## Récapitulatif visuel

```
Avant scroll :
┌──────────── viewport ────────────┐
│ [01 Appel]  [02 Cadrage]  [03.. │  section visible
└──────────────────────────────────┘
 [01]  [02]  [03]  [04]  [spacer]
 ↑ track entier (déborde à droite)

Pendant scroll (section pinée) :
┌──────────── viewport ────────────┐
│  [02 Cadrage]  [03 Création]  [0│  track glisse vers la gauche
└──────────────────────────────────┘

Fin du scroll :
┌──────────── viewport ────────────┐
│  [03 Création]  [04 Livraison]  │  dernier groupe visible + spacer
└──────────────────────────────────┘
```
