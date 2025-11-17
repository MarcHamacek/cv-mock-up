# SCSS Guide & Conventions

Ce document rassemble les conventions, patterns et exemples utilisés dans ce projet pour écrire du SCSS maintenable, efficace et réutilisable.

---

**Structure générale**

- `scss/abstracts/` : variables, maps, mixins.
- `scss/base/` : reset, thème de base.
- `scss/utilities/` : fichiers qui exposent des classes utilitaires.
- `scss/main.scss` : point d'entrée qui `@use`/`@forward` et compile.

---

**Principes clés**

- Toujours utiliser l'API module (`@use`, `@forward`) au lieu de `@import`.
- Ne pas utiliser `@use '... as *'`. Préfixer les symboles via `@use 'abstracts' as abs;` et appeler `abs.$colors` ou `abs.media-breakpoint-up`.
- Stocker tokens (couleurs, spacing, breakpoints) dans des maps SCSS dans `abstracts/variables`.
- Générer les utilitaires en deux phases :
  1. Émettre les classes «globales» (ex. `.m-1`, `.bg-primary`).
  2. Émettre, regroupées par breakpoint, les variantes responsive (`@media` minimales), pour éviter que des règles globales n'écrasent des variantes.
- Préférer des helpers générateurs (ex. `gen-prop`, `gen-prop-two`) pour éviter la duplication à travers les mixins.

---

**Fichiers importants & conventions**

- `scss/main.scss` :

  - `@use 'abstracts' as abs;`
  - `@use 'base/base' as base;`
  - `@use 'utilities/index' as utils;`
  - Ordre : variables -> mixins -> base -> utilities -> components

- Nommage :
  - Variables : `abs.$colors`, `abs.$spacing`, `abs.$grid-breakpoints`.
  - Mixins : `@mixin abs.generate-colors(...)` ou `@mixin abs.media-breakpoint-up($name)`.
  - Classes utilitaires : `.{property}-{token}` et responsive `.{property}-{token}-{bp}` (ex. `.bg-sky-100-md`).

---

**Commandes utiles**

- Installer dépendances (si besoin) :

```bash
npm install
```

- Compiler Sass (npm script déjà présent) :

```bash
npm run sass

npm run sass:watch
```

- Linter SCSS :

```bash
# lancer la vérification
npm run lint:css

# corriger les erreurs auto-fixables
npm run lint:css:fix
```

---

**Stylelint & règles utiles**
Le projet utilise Stylelint + `stylelint-scss`. Quelques règles notables dans `.stylelintrc.json` :

- `customSyntax: 'postcss-scss'` pour parser le SCSS.
- `at-rule-no-unknown` ajusté pour autoriser les at-rules SCSS.
- `media-query-no-invalid` désactivée pour autoriser l'usage de variables dans les media queries.

---

**Pattern: génération d'utilitaires (deux-phase)**

Le repo contient `scss/utilities/_generator.scss` avec des helpers `gen-prop` et `gen-prop-two` (voir exemples ci-dessous).

But : assurer que les règles globales ne réécrasent pas les variantes responsive.

Idée :

1. Dans un mixin, d'abord itérer les maps pour émettre les classes de base.
2. Puis, itérer les breakpoints et, à l'intérieur d'un `@media` générique (via `abs.media-breakpoint-up($bp)`), émettre les variantes correspondant à ce breakpoint.

Exemple conceptuel :

```scss
// 1) classes de base
@include gen-prop($map: $colors, $prefix: 'bg', $property: 'background-color');

// 2) variantes responsive regroupées par breakpoint
@each $bp-name, $bp-value in $breakpoints {
  @include abs.media-breakpoint-up($bp-name) {
    @include gen-prop(
      $map: $colors,
      $prefix: 'bg',
      $property: 'background-color',
      $breakpoints: (
        $bp-name,
      )
    );
  }
}
```
