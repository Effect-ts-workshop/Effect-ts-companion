---
sidebar_position: 8
---

# Concurrence

Effect est construit autour des **Fibers** — des threads virtuels légers qui permettent une concurrence efficace sans les complexités des threads OS.

## Fibers

Une **Fiber** est l'unité d'exécution d'Effect. La distinction clé :

- Un `Effect` = une **description** (lazy, pas encore exécutée)
- Une `Fiber` = une **exécution** en cours

```typescript
import { Effect, Fiber } from "effect"

// Type : Fiber<A, E>  (pas de R — déjà en cours d'exécution)
```

### `Effect.fork` — lancer un Effect en parallèle

```typescript
const program = Effect.gen(function* () {
  // Lance l'effet en arrière-plan, retourne immédiatement une Fiber
  const fiber = yield* Effect.fork(longRunningTask)

  // Continue pendant que longRunningTask tourne
  yield* doSomethingElse

  // Attend le résultat de la fiber
  const result = yield* Fiber.join(fiber)
  return result
})
```

### `Fiber.join` vs `Fiber.await`

```typescript
// join : retourne A ou propage l'erreur E
const value = yield* Fiber.join(fiber)
// Type : Effect<A, E, never>

// await : retourne toujours un Exit, sans propager l'erreur
const exit = yield* Fiber.await(fiber)
// Type : Effect<Exit<A, E>, never, never>
```

### Interrompre une Fiber

```typescript
const program = Effect.gen(function* () {
  const fiber = yield* Effect.fork(neverEndingTask)

  yield* Effect.sleep("1 second")

  // Interrompre proprement
  yield* Fiber.interrupt(fiber)
})
```

## Patterns de concurrence

### `Effect.all` — exécuter en parallèle

```typescript
import { Effect } from "effect"

// Séquentiel (par défaut)
const sequential = Effect.all([fetchUser, fetchPosts, fetchComments])

// Parallèle
const parallel = Effect.all(
  [fetchUser, fetchPosts, fetchComments],
  { concurrency: "unbounded" }
)

// Parallèle limité (max 2 à la fois)
const limited = Effect.all(ids.map(fetchUser), { concurrency: 2 })
```

Si **un** Effect échoue, tous les autres sont interrompus.

### `Effect.race` — le premier qui répond gagne

```typescript
const result = yield* Effect.race(
  fetchFromPrimaryServer,
  fetchFromBackupServer
)
// Retourne le résultat du premier terminé, interrompt l'autre
```

### `Effect.raceAll` — parmi plusieurs

```typescript
const result = yield* Effect.raceAll([
  fetchFromServer1,
  fetchFromServer2,
  fetchFromServer3
])
```

## `Effect.zip` — combiner deux Effects

```typescript
// Séquentiel
const [user, settings] = yield* Effect.zip(fetchUser(id), fetchSettings(id))

// Parallèle
const [user, settings] = yield* Effect.zip(
  fetchUser(id),
  fetchSettings(id),
  { concurrent: true }
)
```

## Exemple : traitement concurrent d'une liste

```typescript
const processUsers = (userIds: string[]) =>
  Effect.gen(function* () {
    const results = yield* Effect.all(
      userIds.map(id =>
        Effect.gen(function* () {
          const user = yield* fetchUser(id)
          const enriched = yield* enrichUserData(user)
          return enriched
        })
      ),
      { concurrency: 5 } // 5 utilisateurs traités simultanément
    )
    return results
  })
```

## Récapitulatif

| Pattern | Usage |
|---|---|
| `Effect.fork` + `Fiber.join` | Parallélisme manuel |
| `Effect.all({ concurrency })` | Parallélisme sur une liste |
| `Effect.race` | Premier résultat disponible |
| `Fiber.interrupt` | Annulation propre |

:::tip Ressources
- [Fibers](https://effect.website/docs/concurrency/fibers/)
- [Concurrency Options](https://effect.website/docs/concurrency/)
:::
