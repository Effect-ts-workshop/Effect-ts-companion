---
sidebar_position: 1
---

# Pourquoi Effect ?

## The Effect Pattern : `Effect<A, E, R>`

Un `Effect<A, E, R>` est une description lazy d'un programme. Il répond à trois questions :

```
Effect<A, E, R>
       │  │  │
       │  │  └── AVEC QUOI ?  → les dépendances nécessaires (Requirements)
       │  └───── ET SI ?      → les erreurs possibles (Error)
       └──────── QUOI ?       → le résultat en cas de succès (success type)
```

:::info Lazy, pas eager
Un `Effect` ne fait rien tant qu'on ne l'exécute pas. Contrairement à une `Promise` qui démarre immédiatement, un `Effect` est une recette — pas un plat déjà cuisiné.
:::

## Comparaison avec une Promise

| | `Promise<A>` | `Effect<A, E, R>` |
|---|---|---|
| Erreurs | non typées (`unknown`) | typées dans `E` |
| Exécution | immédiate (eager) | différée (lazy) |
| Dépendances | implicites | déclarées dans `R` |
| Annulation | via `AbortController` | intégrée |
| Rejouable | non | oui |

## Pourquoi "A" pour le succès ?

Effect est directement inspiré de ZIO (Scala). Cette convention vient des langages fonctionnels :

| Lib | Type | A = |
|---|---|---|
| Haskell | `Either a e` | valeur de succès |
| ZIO | `ZIO[R, E, A]` | valeur de succès |
| fp-ts | `TaskEither<E, A>` | valeur de succès |
| **Effect** | `Effect<A, E, R>` | valeur de succès |

:::tip Ressources

- [The Effect Type](https://effect.website/docs/getting-started/the-effect-type/)
- [Why Effect?](https://effect.website/docs/getting-started/introduction/)

:::
