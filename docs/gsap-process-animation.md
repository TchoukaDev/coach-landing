# Animation — Section Process (scroll horizontal)

> Implémentée en vanilla JS + CSS sticky. GSAP a été tenté puis abandonné —
> ScrollTrigger ajoute une couche d'abstraction qui complique le debug sans
> apporter de valeur réelle sur un effet aussi ciblé. Le vanilla est ici
> plus prévisible et plus facile à maintenir.

---

## Principe général

Le scroll vertical est "converti" en déplacement horizontal des cartes.
L'utilisateur scrolle normalement vers le bas — les cartes défilent vers la gauche.

Deux mécanismes CSS rendent ça possible :

1. **`position: sticky`** — la section reste collée en haut du viewport pendant
   que l'utilisateur scrolle à l'intérieur du wrapper
2. **`translateX`** appliqué par JS — les cartes se déplacent horizontalement
   en fonction de la progression du scroll dans le wrapper

---

## Structure HTML

```
<div id="process-wrapper">          ← hauteur dynamique (JS), donne la "matière à scroller"
  <section id="process-section">    ← sticky top-0, h-screen → reste collée pendant le scroll
    <ol id="process-track">         ← translateX appliqué par JS
      <li> carte 01 </li>
      <li> carte 02 </li>
      <li> carte 03 </li>
      <li> carte 04 </li>
      <li aria-hidden> </li>        ← spacer droit (évite que la dernière carte soit au bord)
    </ol>
  </section>
</div>
```

**Pourquoi un wrapper séparé de la section ?**

La section est `sticky` — elle ne contribue plus à la hauteur du flux normal.
Le wrapper lui donne une "zone d'action" : tant que le scroll est dans le wrapper,
la section reste collée. Quand le scroll dépasse le wrapper, la section se décolle
et reprend son comportement normal.

---

## Calcul JS — étape par étape

```js
const maxTranslate = track.scrollWidth - window.innerWidth + 48;
```

- `track.scrollWidth` = largeur totale du track (toutes les cartes + gaps)
- `window.innerWidth` = ce qui est visible
- La différence = ce qu'il faut faire défiler pour tout voir
- `+ 48` = padding droit (évite que la dernière carte arrive pile au bord)

```js
wrapper.style.height = `calc(100vh + ${maxTranslate}px)`;
```

Le wrapper doit être assez haut pour "consommer" le scroll nécessaire.
- `100vh` = la section elle-même
- `+ maxTranslate` = les pixels de scroll supplémentaires pour traverser toutes les cartes

La section se décolle exactement quand `progress = 1` (scroll terminé).

```js
const wrapperTop = wrapper.offsetTop;
```

`offsetTop` = position statique du wrapper depuis le haut de la page.
**Utiliser `getBoundingClientRect().top` serait une erreur** : cette valeur change
à chaque pixel scrollé, rendant le calcul instable.

```js
const scrollable = wrapper.offsetHeight - window.innerHeight;
```

La plage de `scrollY` où la section est active :
- `wrapper.offsetHeight` = hauteur totale du wrapper (= `100vh + maxTranslate`)
- `- window.innerHeight` = on enlève la hauteur de la section elle-même

Résultat : `scrollable = maxTranslate`. La plage active = exactement la distance
horizontale à parcourir. Ce n'est pas une coïncidence — c'est pourquoi on dimensionne
le wrapper avec `calc(100vh + ${maxTranslate}px)`.

```js
const scrolled = window.scrollY - wrapperTop;
const progress = Math.max(0, Math.min(1, scrolled / scrollable));
```

- `scrolled` = combien on a scrollé depuis l'entrée dans le wrapper
- `progress` = valeur entre 0 et 1 (clampée pour éviter les débordements)

```js
track.style.transform = `translateX(${-progress * maxTranslate}px)`;
```

- `progress = 0` → `translateX(0)` → cartes à leur position initiale
- `progress = 1` → `translateX(-maxTranslate)` → dernière carte visible

---

## Cas particulier : grand écran

```js
if (maxTranslate <= 48) {
  wrapper.style.height = "";
  track.style.transform = "";
  track.style.justifyContent = "center";
  return;
}
```

Quand toutes les cartes tiennent dans le viewport (ex: écran 2K), `maxTranslate`
est nul ou négatif. Aucun scroll nécessaire — les cartes sont centrées et
l'animation est désactivée.

---

## Comportement au resize

`update()` est appelée à chaque `resize`. Elle recalcule :
- `maxTranslate` (les cartes ont peut-être changé de taille)
- `wrapper.style.height`
- La position courante des cartes

Pas besoin d'un cleanup : `transform` et `height` sont simplement écrasés.

---

## Récapitulatif visuel

```
État initial (scroll = 0 dans le wrapper) :
┌──────────── viewport ────────────┐
│ [01 Appel]  [02 Cadrage]  [03.. │
└──────────────────────────────────┘
 [01]  [02]  [03]  [04]  [· · ·]
         track complet → déborde à droite

Milieu (progress = 0.5) :
┌──────────── viewport ────────────┐
│   [02 Cadrage]  [03 Création]   │
└──────────────────────────────────┘

Fin (progress = 1) :
┌──────────── viewport ────────────┐
│  [03 Création]  [04 Livraison]  │
└──────────────────────────────────┘
                         [spacer]→|
```

---

## Pourquoi pas GSAP ?

GSAP ScrollTrigger est puissant pour des animations complexes (timelines,
pinning de plusieurs éléments, séquençage). Ici, l'effet se résume à un
mapping `scrollY → translateX`. Vanilla suffit amplement et évite :

- Une dépendance externe (~60kb)
- La couche d'abstraction de ScrollTrigger qui masque ce qui se passe vraiment
- Les conflits potentiels avec `matchMedia` et le resize handling
