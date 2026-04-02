
Effect ist eine TypeScript-Library für **funktionale, typsichere Fehlerbehandlung und Nebeneffekte**.

Das Kernprinzip: Statt Dinge einfach *auszuführen*, **beschreibst** du zuerst, was passieren soll – und führst es dann kontrolliert aus. Das nennt man **lazy evaluation**.

## Das zentrale Konzept: `Effect<A, E, R>`

```typescript

Effect<A, E, R>

// │ │ └── Requirements (Abhängigkeiten, z.B. ein Service)

// │ └───── Error (was kann schiefgehen?)

// └──────── Success (was kommt raus bei Erfolg?)

```


Ein `Effect<number, Error, never>` bedeutet:

- gibt bei Erfolg eine `number` zurück

- kann mit `Error` fehlschlagen

- braucht keine externen Abhängigkeiten

## Warum Effect statt try/catch?

```typescript

// Ohne Effect: Fehler sind unsichtbar im Typ

async function fetchUser(id: string): Promise<User> {

// Kann werfen – aber der Typ sagt das nicht!

const res = await fetch(`/users/${id}`);

return res.json();

}

  

// Mit Effect: Fehler sind im Typ sichtbar

const fetchUser = (id: string): Effect.Effect<User, HttpError, never> =>

Effect.tryPromise({

try: () => fetch(`/users/${id}`).then(r => r.json()),

catch: (e) => new HttpError(e)

});

```

  

## Grundlegende Operationen
  

```typescript

import { Effect } from "effect";

  

// Einen Wert wrappen

const succeed = Effect.succeed(42);

const fail = Effect.fail(new Error("oops"));

  

// Transformieren

const doubled = Effect.map(succeed, n => n * 2);

  

// Verketten (flatMap)

const result = Effect.flatMap(succeed, n =>

n > 0 ? Effect.succeed("positive") : Effect.fail("negative")

);

  

// Fehler behandeln

const handled = Effect.catchAll(fail, err =>

Effect.succeed(`Fallback: ${err.message}`)

);

```

  

## Effect.gen – der "async/await" von Effect
  

```typescript

// Ohne gen (callback-heavy):

const program = Effect.flatMap(getUser, user =>

Effect.flatMap(getOrders(user.id), orders =>

Effect.map(calcTotal(orders), total => ({ user, total }))

)

);

  

// Mit gen (lesbar wie async/await):

const program = Effect.gen(function* () {

const user = yield* getUser;

const orders = yield* getOrders(user.id);

const total = yield* calcTotal(orders);

return { user, total };

});

```

  

## Layer & Dependency Injection


Effect hat ein eingebautes DI-System über **Layers**:


```typescript

// Service definieren

class Database extends Effect.Service<Database>()("Database", {

effect: Effect.succeed({

query: (sql: string) => Effect.succeed([])

})

}) {}

  

// Service verwenden

const program = Effect.gen(function* () {

const db = yield* Database;

const rows = yield* db.query("SELECT * FROM users");

return rows;

});

  

// Layer bereitstellen und ausführen

Effect.runPromise(program.pipe(Effect.provide(Database.Default)));

```

  

## Ausführen (am Ende der Welt)
  

```typescript

// Promise zurückgeben

Effect.runPromise(program);

  

// Synchron (nur für pure Effects)

Effect.runSync(program);

  

// Mit NodeRuntime (für Server-Apps)

import { NodeRuntime } from "@effect/platform-node";

NodeRuntime.runMain(program);

```

  

## Wann Effect sinnvoll ist

✅ Gut für:

- Server-Apps mit vielen Fehlerquellen (DB, HTTP, Filesystem)

- Code der testbar und austauschbar sein soll (via Layers)

- Wenn du Fehlertypen explizit im Code haben willst


❌ Weniger sinnvoll für:

- Kleine Scripts (zu viel Overhead)

- Teams die nicht mit FP vertraut sind


## Verwandte Konzepte


- `pipe` – Funktionen verketten: `value.pipe(f, g, h)` = `h(g(f(value)))`

- `Schema` – Validierung und Parsing (ähnlich wie Zod, aber in Effect integriert)

- `Stream` – für kontinuierliche Datenströme

- `Fiber` – leichtgewichtige Threads für Concurrency