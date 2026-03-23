---
sidebar_position: 1
---

# Bienvenue dans l'atelier Effect.ts

Cet atelier de **3 heures** vous fait découvrir [Effect.ts](https://effect.website) — une bibliothèque TypeScript pour construire des applications robustes, composables et maintenables.

Pas de prérequis fonctionnel requis : si vous écrivez du TypeScript, vous êtes au bon endroit.

---

## Ce que vous allez apprendre

À l'issue de cet atelier, vous saurez :

- Modéliser des succès et des erreurs avec le type `Effect<A, E, R>`
- Composer des effets avec `pipe` et `Effect.gen`
- Gérer les erreurs de façon explicite et typée
- Structurer une application avec des **Services** et des **Layers**

---

## Déroulé des 3 heures

| #   | Thème                                | Durée  |
| --- | ------------------------------------ | ------ |
| 0   | Setup & prise en main                | 10 min |
| 1   | Premiers Effects — créer et exécuter | 20 min |
| 2   | Composer avec `pipe` et `Effect.gen` | 30 min |
| 3   | Gestion des erreurs                  | 30 min |
| 4   | Services et Layers                   | 40 min |
| 5   | Bonus : ressources, concurrence      | 30 min |
| —   | Questions / démo finale              | 20 min |

---

## Setup

### Prérequis

- Node.js >= 20
- Un éditeur avec support TypeScript (VS Code recommandé)

### Cloner le repo d'exercices

```bash
git clone git@github.com:Effect-ts-workshop/Effect-ts-workshop.git
cd <Effect-ts-workshop>
npm install
```

### Vérifier que tout fonctionne

```bash

npm run check
```

Vous devriez voir les exercices listés sans erreur de compilation.

---

## Comment utiliser ce site

- La section **Exercices** contient les énoncés, dans l'ordre de l'atelier.
- La section **Documentation** est votre référence : concepts, exemples TypeScript, tableaux récap.
- Chaque exercice renvoie vers la page de doc correspondante si vous avez besoin d'un rappel.

:::tip Doc officielle
La documentation complète d'Effect.ts est disponible sur [effect.website/docs](https://effect.website/docs).
:::
