---
sidebar_position: 5
---

# Gestion des erreurs

Effect distingue deux catégories d'erreurs, ce qui permet une gestion précise et typée.

## Erreurs attendues vs inattendues

### Erreurs attendues (`E`)

Ce sont les erreurs **prévisibles** de ton domaine métier. Elles apparaissent dans le type de l'Effect et sont récupérables.

```typescript
// L'erreur NotFoundError est déclarée dans le type
const findUser = (id: string): Effect.Effect<User, NotFoundError> =>
  Effect.fail(new NotFoundError(id))
```

### Erreurs inattendues (Defects)

Ce sont les erreurs **non prévues** (bug, crash, assertion). Elles ne figurent pas dans le type `E` mais sont capturées par le runtime.

```typescript
// Aucune erreur déclarée, mais le code peut planter
const risky = Effect.sync(() => {
  throw new Error("Unexpected crash!")
})
// Type : Effect<never, never, never> — mais peut quand même échouer
```

:::info Analogie
Les erreurs attendues = exceptions vérifiées (Java checked exceptions)
Les defects = exceptions non vérifiées (RuntimeException)
:::

## Récupérer des erreurs

### `catchAll` — attraper toutes les erreurs

```typescript
const safe = program.pipe(
  Effect.catchAll(error =>
    Effect.succeed(`Erreur récupérée : ${error.message}`)
  )
)
```

### `catchTag` — attraper par type d'erreur

Nécessite que l'erreur ait un champ `_tag`.

```typescript
const safe = program.pipe(
  Effect.catchTag("NotFoundError", (error) =>
    Effect.succeed(`Utilisateur ${error.id} introuvable`)
  )
)
```

### `catchTags` — gérer plusieurs types d'erreurs

```typescript
const safe = program.pipe(
  Effect.catchTags({
    NotFoundError: (e) => Effect.succeed(`Not found: ${e.id}`),
    ValidationError: (e) => Effect.succeed(`Invalid: ${e.message}`)
  })
)
```

### `match` — gérer succès et erreur en même temps

```typescript
const result = program.pipe(
  Effect.match({
    onFailure: (error) => `Erreur: ${error.message}`,
    onSuccess: (value) => `Succès: ${value}`
  })
)
// Type : Effect<string, never, never>
```

## Transformer les erreurs

### `mapError` — transformer le type d'erreur

```typescript
const mapped = program.pipe(
  Effect.mapError(e => new AppError(`Wrapped: ${e.message}`))
)
```

## Retry et timeout

### `Effect.retry` — relancer en cas d'erreur

```typescript
import { Effect, Schedule } from "effect"

const withRetry = program.pipe(
  Effect.retry(Schedule.recurs(3)) // 3 tentatives
)

// Avec délai exponentiel
const withBackoff = program.pipe(
  Effect.retry(Schedule.exponential("100 millis"))
)
```

### `Effect.timeout` — limiter la durée

```typescript
const withTimeout = program.pipe(
  Effect.timeout("5 seconds")
)
// Type : Effect<A, E | TimeoutException, R>
```

## Récapitulatif

| Opérateur | Utilisation |
|---|---|
| `catchAll` | Récupère toutes les erreurs `E` |
| `catchTag` | Récupère une erreur par son `_tag` |
| `catchTags` | Récupère plusieurs erreurs par tag |
| `match` | Gère succès et erreur ensemble |
| `mapError` | Transforme le type d'erreur |
| `retry` | Relance sur erreur (avec Schedule) |
| `timeout` | Limite la durée d'exécution |

:::tip Ressources
- [Two Error Types](https://effect.website/docs/error-management/two-error-types/)
- [Expected Errors](https://effect.website/docs/error-management/expected-errors/)
- [Retrying](https://effect.website/docs/error-management/retrying/)
:::
