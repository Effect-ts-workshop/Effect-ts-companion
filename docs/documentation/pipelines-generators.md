---
sidebar_position: 4
---

# Pipelines et Generators

Effect propose deux styles pour composer des Effects. Les deux sont valides — choisir l'un ou l'autre dépend du contexte et des préférences de l'équipe.

## Style 1 : `pipe` et opérateurs

`pipe` permet d'enchaîner des transformations de gauche à droite, à la manière d'un pipeline Unix.

```typescript
import { Effect, pipe } from "effect"

const program = pipe(
  Effect.succeed(5),
  Effect.map(n => n * 2),
  Effect.flatMap(n => Effect.succeed(n + 1)),
  Effect.map(n => `Result: ${n}`)
)
// Effect<string, never, never>
```

La plupart des fonctions Effect sont aussi disponibles en méthode via `.pipe()` :

```typescript
const program = Effect.succeed(5).pipe(
  Effect.map(n => n * 2),
  Effect.flatMap(n => Effect.succeed(n + 1))
)
```

### Opérateurs courants

| Opérateur | Rôle |
|---|---|
| `Effect.map(f)` | Transforme la valeur de succès |
| `Effect.flatMap(f)` | Transforme en retournant un nouvel Effect |
| `Effect.tap(f)` | Effet de bord sans changer la valeur |
| `Effect.andThen(f)` | Enchaîne un autre Effect |
| `Effect.mapError(f)` | Transforme l'erreur |

## Style 2 : `Effect.gen` et `yield*`

`Effect.gen` utilise les generators JavaScript pour obtenir une syntaxe proche de `async/await`. Plus lisible pour les séquences longues.

```typescript
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const a = yield* Effect.succeed(5)
  const b = yield* Effect.succeed(10)
  return a + b
})
// Effect<number, never, never>
```

Le `yield*` "déballe" l'Effect et retourne sa valeur de succès — ou court-circuite en cas d'erreur.

### Avec du contrôle de flux

```typescript
const findAndGreet = (id: string) =>
  Effect.gen(function* () {
    const user = yield* findUser(id)

    if (!user) {
      return yield* Effect.fail(new NotFoundError(id))
    }

    const greeting = yield* buildGreeting(user)
    return greeting
  })
```

### Avec des boucles

```typescript
const processAll = (ids: string[]) =>
  Effect.gen(function* () {
    const results = []
    for (const id of ids) {
      const result = yield* processOne(id)
      results.push(result)
    }
    return results
  })
```

## Comparaison des deux styles

```typescript
// Style pipe
const program = pipe(
  fetchUser(id),
  Effect.flatMap(user => fetchPosts(user.id)),
  Effect.map(posts => posts.length)
)

// Style gen — équivalent
const program = Effect.gen(function* () {
  const user = yield* fetchUser(id)
  const posts = yield* fetchPosts(user.id)
  return posts.length
})
```

| | `pipe` | `Effect.gen` |
|---|---|---|
| Lisibilité séquentielle | correcte | excellente |
| Intégration control flow | difficile | naturelle |
| Débogage | moins intuitif | plus proche de JS classique |
| Style | fonctionnel pur | impératif/fonctionnel |

:::tip Ressources
- [Using Generators](https://effect.website/docs/getting-started/using-generators/)
- [Pipeline](https://effect.website/docs/getting-started/building-pipelines/)
:::
