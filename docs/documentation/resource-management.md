---
sidebar_position: 7
---

# Gestion des ressources

Une ressource est tout ce qui nécessite une **acquisition** (connexion, fichier, lock) et une **libération** (fermeture, release). Effect garantit que le nettoyage s'exécute **toujours**, même en cas d'erreur ou d'interruption.

## Le problème

```typescript
// ❌ Sans Effect : la connexion peut ne jamais être fermée
const conn = await db.connect()
const result = await conn.query("SELECT ...") // si ça plante ici...
await conn.close() // ...cette ligne n'est jamais atteinte
```

## `Effect.acquireRelease`

Le pattern fondamental pour gérer les ressources :

```typescript
import { Effect } from "effect"

const dbConnection = Effect.acquireRelease(
  // Acquire : comment obtenir la ressource
  Effect.tryPromise(() => db.connect()),

  // Release : comment la libérer (toujours exécuté)
  (conn, exit) => Effect.promise(() => conn.close())
)
// Type : Effect<Connection, Error, Scope>
```

Le second paramètre reçoit un `Exit` qui indique si l'Effect s'est terminé avec succès ou non :

```typescript
(conn, exit) => Effect.sync(() => {
  if (Exit.isFailure(exit)) {
    console.log("Closing after error")
  }
  conn.close()
})
```

## `Effect.scoped` — délimiter la durée de vie

`acquireRelease` retourne un `Effect<A, E, Scope>`. `Effect.scoped` gère le cycle de vie du scope automatiquement :

```typescript
const program = Effect.scoped(
  Effect.gen(function* () {
    const conn = yield* dbConnection  // acquis ici
    const rows = yield* conn.query("SELECT * FROM users")
    return rows
    // conn.close() appelé automatiquement en sortant du scoped
  })
)
// Type : Effect<Row[], Error, never>  ← Scope retiré du R
```

## `Effect.addFinalizer` — cleanup ad hoc

Pour ajouter un nettoyage à l'intérieur d'un scope existant :

```typescript
const program = Effect.scoped(
  Effect.gen(function* () {
    yield* Effect.addFinalizer((exit) =>
      Effect.sync(() => console.log(`Cleanup, exit: ${exit._tag}`))
    )

    // ... logique du programme
    return "done"
  })
)
```

## Exemple complet

```typescript
import { Effect, Exit } from "effect"

class DatabaseConnection {
  static open = Effect.acquireRelease(
    Effect.sync(() => {
      console.log("Opening connection")
      return { query: (sql: string) => Effect.succeed([]) }
    }),
    (conn, exit) =>
      Effect.sync(() => console.log(
        Exit.isSuccess(exit) ? "Closing cleanly" : "Closing after error"
      ))
  )
}

const program = Effect.scoped(
  Effect.gen(function* () {
    const db = yield* DatabaseConnection.open
    const users = yield* db.query("SELECT * FROM users")
    return users
  })
)

Effect.runPromise(program)
// Opening connection
// Closing cleanly
```

## Garanties d'Effect

| Situation | Le finalizer s'exécute ? |
|---|---|
| Succès normal | ✅ oui |
| Erreur attendue | ✅ oui |
| Defect (crash) | ✅ oui |
| Interruption du fiber | ✅ oui |

:::tip Ressources
- [Resource Management](https://effect.website/docs/resource-management/)
- [Scope](https://effect.website/docs/resource-management/scope/)
:::
