---
sidebar_position: 3
---

# Exécuter des Effects

Un `Effect` est une description — pour qu'il se passe quelque chose, il faut l'exécuter. Les fonctions `run*` sont les points de sortie du monde Effect vers le monde réel.

:::warning Une seule fois, en haut de la pile
Les fonctions `run*` doivent être appelées le plus tard possible, idéalement une seule fois à l'entrée de l'application. Tout le reste reste dans le monde `Effect`.
:::

## `Effect.runSync` — exécution synchrone

À utiliser quand l'Effect est entièrement synchrone.

```typescript
import { Effect } from "effect"

const result = Effect.runSync(Effect.succeed(42))
console.log(result) // 42
```

Lance une exception si l'Effect échoue ou contient du code asynchrone.

## `Effect.runPromise` — exécution asynchrone

La façon la plus courante d'exécuter un Effect dans une application Node.js ou navigateur.

```typescript
Effect.runPromise(Effect.succeed(42)).then(console.log)
// 42

Effect.runPromise(Effect.fail(new Error("oops"))).catch(console.error)
// Error: oops
```

## `Effect.runSyncExit` et `Effect.runPromiseExit` — avec le résultat complet

Ces variantes retournent un `Exit<A, E>` qui capture le résultat de manière explicite sans lever d'exception.

```typescript
import { Effect, Exit } from "effect"

const exit = Effect.runSyncExit(Effect.succeed(1))
// { _id: "Exit", _tag: "Success", value: 1 }

const failExit = Effect.runSyncExit(Effect.fail("oops"))
// { _id: "Exit", _tag: "Failure", cause: ... }

if (Exit.isSuccess(exit)) {
  console.log(exit.value)
}
```

## `Effect.runFork` — exécution en arrière-plan

Lance l'Effect dans une fiber sans attendre le résultat. Retourne une `RuntimeFiber` qu'on peut interrompre.

```typescript
import { Effect, Fiber } from "effect"

const fiber = Effect.runFork(
  Effect.delay(Effect.log("Done!"), "2 seconds")
)

// Plus tard, si besoin d'annuler :
Effect.runFork(Fiber.interrupt(fiber))
```

## Récapitulatif

| Fonction | Retourne | Usage |
|---|---|---|
| `runSync` | `A` (ou throw) | Effects purement synchrones |
| `runSyncExit` | `Exit<A, E>` | Synchrone, sans exception |
| `runPromise` | `Promise<A>` | Le plus courant |
| `runPromiseExit` | `Promise<Exit<A, E>>` | Contrôle fin du résultat |
| `runFork` | `RuntimeFiber<A, E>` | Arrière-plan / fire-and-forget |

:::tip Ressources
- [Running Effects](https://effect.website/docs/getting-started/running-effects/)
:::
