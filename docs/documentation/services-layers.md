---
sidebar_position: 6
---

# Services et Layers

Le troisième paramètre de `Effect<A, E, R>` — le `R` — représente les **dépendances** requises. Effect propose un système d'injection de dépendances intégré, sans framework externe.

## Services

Un **service** est une interface qui expose des fonctionnalités. On le définit avec `Context.Tag`.

```typescript
import { Context, Effect } from "effect"

// 1. Définir l'interface
interface EmailService {
  readonly send: (to: string, body: string) => Effect.Effect<void>
}

// 2. Créer le Tag (identifiant unique du service)
class EmailService extends Context.Tag("EmailService")<
  EmailService,
  { readonly send: (to: string, body: string) => Effect.Effect<void> }
>() {}
```

### Utiliser un service

```typescript
const notifyUser = (userId: string) =>
  Effect.gen(function* () {
    const email = yield* EmailService  // récupère le service depuis le contexte
    yield* email.send(userId, "Bienvenue !")
  })
// Type : Effect<void, never, EmailService>
```

Le `R` indique qu'il faut un `EmailService` pour exécuter cet Effect.

## Layers

Un **Layer** est la "recette" pour construire un service. C'est ce qu'on fournit pour satisfaire le `R`.

```typescript
import { Layer } from "effect"

// Implémentation concrète
const EmailServiceLive = Layer.succeed(EmailService, {
  send: (to, body) =>
    Effect.sync(() => console.log(`Sending to ${to}: ${body}`))
})
```

### Layer avec dépendances

Un Layer peut lui-même dépendre d'autres services :

```typescript
class Logger extends Context.Tag("Logger")<
  Logger,
  { readonly log: (msg: string) => Effect.Effect<void> }
>() {}

const EmailServiceWithLogger = Layer.effect(
  EmailService,
  Effect.gen(function* () {
    const logger = yield* Logger
    return {
      send: (to, body) =>
        Effect.gen(function* () {
          yield* logger.log(`Sending email to ${to}`)
          // ... logique d'envoi
        })
    }
  })
)
// Type : Layer<EmailService, never, Logger>
```

### Composer des Layers

```typescript
// Fusionner des layers indépendants
const AppLayer = Layer.merge(LoggerLive, DatabaseLive)

// Enchaîner des layers dépendants
const FullLayer = EmailServiceWithLogger.pipe(
  Layer.provide(LoggerLive)
)
```

## Fournir les dépendances

### `Effect.provide` — fournir un Layer complet

```typescript
const program = notifyUser("user-123")
// Type : Effect<void, never, EmailService>

const runnable = program.pipe(
  Effect.provide(EmailServiceLive)
)
// Type : Effect<void, never, never> ← prêt à être exécuté
```

### `Effect.provideService` — fournir directement une implémentation

```typescript
const runnable = program.pipe(
  Effect.provideService(EmailService, {
    send: (to, body) => Effect.sync(() => console.log(`${to}: ${body}`))
  })
)
```

## Schéma complet

```
┌─────────────────────────────────────────────┐
│                  Program                    │
│   Effect<void, never, EmailService | Logger>│
└─────────────────────────────────────────────┘
                      │ Effect.provide(AppLayer)
┌─────────────────────────────────────────────┐
│                  AppLayer                   │
│   EmailServiceLive ──── LoggerLive          │
└─────────────────────────────────────────────┘
                      │
              Effect.runPromise(...)
```

:::tip Ressources
- [Services](https://effect.website/docs/requirements-management/services/)
- [Layers](https://effect.website/docs/requirements-management/layers/)
:::
