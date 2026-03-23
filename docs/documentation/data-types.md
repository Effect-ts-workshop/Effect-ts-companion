---
sidebar_position: 9
---

# Types de données utilitaires

Effect fournit plusieurs types de données fonctionnels qui complètent `Effect<A, E, R>`. Ces types permettent d'exprimer l'absence de valeur, les bifurcations, et les causes d'erreur de manière explicite.

## `Option<A>` — valeur optionnelle

`Option` remplace `null` et `undefined` de manière explicite et type-safe.

```typescript
import { Option } from "effect"

type Option<A> = Some<A> | None
```

### Créer un Option

```typescript
const some = Option.some(42)        // Some<number>
const none = Option.none()          // None

// Depuis une valeur nullable
const fromNull = Option.fromNullable(user?.name)
// Option.some("Alice") ou Option.none()
```

### Travailler avec Option

```typescript
// Transformer la valeur si présente
const doubled = Option.map(Option.some(5), n => n * 2)
// Some(10)

// Récupérer avec une valeur par défaut
const value = Option.getOrElse(result, () => "valeur par défaut")

// Chaîner
const result = Option.flatMap(
  Option.fromNullable(user),
  user => Option.fromNullable(user.address)
)
```

### Intégration avec Effect

```typescript
import { Effect, Option } from "effect"

// Option vers Effect
const program = Effect.gen(function* () {
  const maybeUser = yield* findUser(id).pipe(Effect.option)
  // maybeUser : Option<User>

  if (Option.isNone(maybeUser)) {
    return yield* Effect.fail(new NotFoundError(id))
  }

  return maybeUser.value
})
```

## `Either<R, L>` — succès ou échec explicite

`Either` représente une valeur qui peut être de deux types. Par convention : `Right` = succès, `Left` = erreur.

```typescript
import { Either } from "effect"

type Either<R, L> = Right<R> | Left<L>
```

### Créer un Either

```typescript
const right = Either.right(42)       // Right<number>
const left = Either.left("erreur")   // Left<string>
```

### Travailler avec Either

```typescript
// Match les deux cas
const message = Either.match(result, {
  onLeft: (err) => `Erreur: ${err}`,
  onRight: (val) => `Valeur: ${val}`
})

// Transformer le Right
const doubled = Either.map(Either.right(5), n => n * 2)
// Right(10)
```

### Intégration avec Effect

```typescript
// Either vers Effect
const effect = Effect.fromEither(Either.right(42))

// Effect vers Either (ne peut plus échouer)
const program = someEffect.pipe(Effect.either)
// Effect<Either<A, E>, never, R>
```

## `Cause<E>` — cause d'erreur enrichie

`Cause` capture le **pourquoi** d'un échec avec plus de précision qu'une simple erreur.

```typescript
import { Cause } from "effect"
```

### Variantes de Cause

| Variante | Description |
|---|---|
| `Cause.fail(e)` | Erreur attendue (dans `E`) |
| `Cause.die(defect)` | Erreur inattendue / crash |
| `Cause.interrupt(fiberId)` | Fiber interrompue |
| `Cause.sequential(c1, c2)` | Deux causes en séquence |
| `Cause.parallel(c1, c2)` | Deux causes en parallèle |

### Inspecter une Cause

```typescript
const program = Effect.gen(function* () {
  const exit = yield* Effect.runPromiseExit(riskyEffect)

  if (Exit.isFailure(exit)) {
    const cause = exit.cause

    if (Cause.isFailType(cause)) {
      console.log("Expected error:", cause.error)
    } else if (Cause.isDieType(cause)) {
      console.log("Unexpected crash:", cause.defect)
    }
  }
})
```

## Récapitulatif

| Type | Usage |
|---|---|
| `Option<A>` | Valeur présente ou absente |
| `Either<R, L>` | Deux résultats possibles (succès/erreur) |
| `Cause<E>` | Cause enrichie d'un échec Effect |

:::tip Ressources
- [Option](https://effect.website/docs/data-types/option/)
- [Either](https://effect.website/docs/data-types/either/)
- [Cause](https://effect.website/docs/data-types/cause/)
:::
