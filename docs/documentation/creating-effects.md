---
sidebar_position: 2
---

# Créer des Effects

Avant d'exécuter quoi que ce soit, il faut construire un `Effect`. Voici les constructeurs essentiels.

## Valeurs pures

### `Effect.succeed` — une valeur garantie

```typescript
import { Effect } from "effect"

const hello = Effect.succeed("Hello, World!")
// Type : Effect<string, never, never>
```

### `Effect.fail` — une erreur attendue

```typescript
const error = Effect.fail(new Error("Something went wrong"))
// Type : Effect<never, Error, never>
```

:::info `never` dans le type
`never` signifie "impossible". Un `Effect<string, never, never>` ne peut pas échouer et n'a pas de dépendances.
:::

## Code synchrone

### `Effect.sync` — effet synchrone sans risque d'erreur

```typescript
const now = Effect.sync(() => Date.now())
// Type : Effect<number, never, never>
```

### `Effect.try` — effet synchrone qui peut lever une exception

```typescript
const parse = (input: string) =>
  Effect.try({
    try: () => JSON.parse(input),
    catch: (e) => new Error(`Parsing failed: ${e}`)
  })
// Type : Effect<unknown, Error, never>
```

Sans `catch`, l'erreur est typée `UnknownException`.

## Code asynchrone

### `Effect.promise` — Promise qui ne peut pas échouer

```typescript
const fetchUser = Effect.promise(() =>
  fetch("https://api.example.com/user").then(r => r.json())
)
// Type : Effect<unknown, never, never>
```

### `Effect.tryPromise` — Promise qui peut échouer

```typescript
const fetchUser = (id: string) =>
  Effect.tryPromise({
    try: () => fetch(`/users/${id}`).then(r => r.json()),
    catch: (e) => new Error(`Fetch failed: ${e}`)
  })
// Type : Effect<unknown, Error, never>
```

## Modéliser les erreurs avec des tags

Une bonne pratique est de créer des classes d'erreur avec un `_tag` pour les identifier facilement :

```typescript
class NotFoundError {
  readonly _tag = "NotFoundError"
  constructor(readonly id: string) {}
}

class ValidationError {
  readonly _tag = "ValidationError"
  constructor(readonly message: string) {}
}

const findUser = (id: string): Effect.Effect<User, NotFoundError> =>
  id === "42"
    ? Effect.succeed({ id, name: "Alice" })
    : Effect.fail(new NotFoundError(id))
```

:::tip Ressources
- [Creating Effects](https://effect.website/docs/getting-started/creating-effects/)
:::
