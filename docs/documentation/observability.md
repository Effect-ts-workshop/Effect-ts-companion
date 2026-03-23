---
sidebar_position: 10
---

# Observabilité

Effect intègre nativement le logging, les métriques et le tracing — sans dépendances externes à configurer.

## Logging

### Niveaux de log

```typescript
import { Effect } from "effect"

Effect.logDebug("Détail de debug")
Effect.logInfo("Information générale")
Effect.logWarning("Attention à ceci")
Effect.logError("Une erreur s'est produite")
Effect.logFatal("Erreur critique")
```

Par défaut, seuls les logs `Info` et au-dessus sont affichés.

### Format de sortie

```
timestamp=2024-01-15T10:30:00.000Z level=INFO fiber=#0 message="Utilisateur créé"
```

Effect enrichit automatiquement chaque log avec :
- l'horodatage
- le niveau
- l'identifiant de la fiber
- la durée du span (si applicable)

### Configurer le niveau minimum

```typescript
import { Effect, Logger, LogLevel } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.logDebug("visible maintenant")
  yield* Effect.logInfo("toujours visible")
}).pipe(
  Logger.withMinimumLogLevel(LogLevel.Debug)
)
```

### Annoter les logs

Ajouter des métadonnées à tous les logs d'un Effect :

```typescript
const program = Effect.gen(function* () {
  yield* Effect.logInfo("Traitement démarré")
  yield* processUser(id)
  yield* Effect.logInfo("Traitement terminé")
}).pipe(
  Effect.annotateLogs({ userId: id, requestId: "req-123" })
)

// Résultat :
// level=INFO message="Traitement démarré" userId=42 requestId=req-123
// level=INFO message="Traitement terminé" userId=42 requestId=req-123
```

### Loggers personnalisés

```typescript
import { Logger } from "effect"

// Formats disponibles
Logger.stringLogger     // texte classique (défaut)
Logger.jsonLogger       // JSON structuré
Logger.prettyLogger     // couleurs pour le terminal
Logger.logfmtLogger     // format logfmt
```

Remplacer le logger par défaut :

```typescript
const program = mainEffect.pipe(
  Effect.provide(Logger.replace(Logger.defaultLogger, Logger.jsonLogger))
)
```

## Tracing

Effect génère automatiquement des **spans** pour les opérations. Idéal pour le debug distribué avec OpenTelemetry.

```typescript
import { Effect } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.logInfo("Démarrage")
  const result = yield* fetchData()
  return result
}).pipe(
  Effect.withSpan("fetchUserData", {
    attributes: { userId: "42" }
  })
)
```

Les spans peuvent être imbriqués et capturent automatiquement la durée et le statut (succès/erreur).

## Métriques

```typescript
import { Metric, Effect } from "effect"

// Compteur
const requestCounter = Metric.counter("http_requests_total")

// Histogramme
const responseTime = Metric.histogram(
  "http_response_time_ms",
  Metric.Histogram.Boundaries.linear({ start: 0, width: 100, count: 10 })
)

// Utilisation
const program = Effect.gen(function* () {
  yield* Metric.increment(requestCounter)
  const start = Date.now()
  const result = yield* handleRequest()
  yield* Metric.record(responseTime, Date.now() - start)
  return result
})
```

## Récapitulatif

| Fonctionnalité | API clé |
|---|---|
| Logs par niveau | `Effect.logDebug/Info/Warning/Error` |
| Filtrer les niveaux | `Logger.withMinimumLogLevel` |
| Métadonnées | `Effect.annotateLogs` |
| Format JSON | `Logger.replace(Logger.jsonLogger)` |
| Spans / Tracing | `Effect.withSpan` |
| Compteurs / Histogrammes | `Metric.counter`, `Metric.histogram` |

:::tip Ressources
- [Logging](https://effect.website/docs/observability/logging/)
- [Tracing](https://effect.website/docs/observability/tracing/)
- [Metrics](https://effect.website/docs/observability/metrics/)
:::
