
Gute Programmierpraxis ist der Verzicht auf globale oder statische Variablen. Denn schon in einem Single-Threaded-Programm können diese, wenn unkontrolliert verwendet, zu unerwarteten Side Effects führen. 

**Beispiel:** Funktion A und Funktion B verwenden beide die globale Variable `count = 5` und erhöhen sie jeweils um 1. Erwartet wird das Endergebnis 7 – doch wenn Funktion B den Wert liest, bevor Funktion A ihren aktualisierten Wert zurückgeschrieben hat, rechnen beide mit 5 und das Ergebnis ist 6.

Noch schwieriger zu kontrollieren ist der Zustand in einem Multithreading-Programm. Wenn viele Operationen parallel laufen und dabei auf dieselbe globale Variable zugreifen, hängt das Ergebnis entscheidend von der Reihenfolge der Threads ab. Man nennt dies [[race condition]]. das Programm wird dadurch unvorhersehbar.

Und hier kommen Monads ins Spiel. Sie versuchen nicht side effects oder globalen state zu verhindern. Sondern sie versuchen ihn zu kontrollieren indem sie das Typ System verwenden. Hier ueberprueft der compiler die Richtigkeit der Monaden. 


Eine Monade ist ein **Entwurfsmuster aus der funktionalen Programmierung**, das es erlaubt, Berechnungen zu verketten, die einen bestimmten Kontext tragen – z.B. einen möglichen Fehler, einen optionalen Wert oder einen Nebeneffekt.


Technisch gesehen ist eine Monade eine **Typklasse** mit drei Dingen:

1. Einem **Typ-Konstruktor** `M<A>` – verpackt einen Wert in einen Kontext

2. **`of` / `return`** – packt einen Wert in die Monade: `A → M<A>`

3. **`flatMap` / `bind` / `chain`** – verkettet Berechnungen: `M<A> → (A → M<B>) → M<B>`

  

## Die Kernidee: Werte in einem Kontext verketten


```typescript

// Ohne Monade: Fehlerbehandlung verschmutzt die Logik

function getCity(userId: string): string | null {

const user = findUser(userId);

if (!user) return null;

const address = getAddress(user);

if (!address) return null;

return address.city;

}

  

// Mit Option-Monade: sauber verkettet

const getCity = (userId: string): Option<string> =>

findUser(userId)

.flatMap(user => getAddress(user))

.map(address => address.city);

```

  

## Die Monadengesetze

Jede echte Monade muss drei Gesetze erfüllen:

```typescript

// 1. Left Identity: of(a).flatMap(f) === f(a)

Option.some(5).flatMap(n => Option.some(n * 2))

// === Option.some(10)

  

// 2. Right Identity: m.flatMap(of) === m

Option.some(5).flatMap(Option.some)

// === Option.some(5)

  

// 3. Associativity: m.flatMap(f).flatMap(g) === m.flatMap(x => f(x).flatMap(g))

// Die Reihenfolge der Verkettung ist egal, solange die Reihenfolge der Operationen gleich bleibt

```

  

## Häufige Monaden in der Praxis

### Option / Maybe – optionale Werte

```typescript

// Kein null-check nötig, der Kontext trägt "könnte leer sein"

type Option<A> = Some<A> | None

  

Option.some(42)

.map(n => n * 2) // Some(84)

.flatMap(n => n > 100

? Option.none()

: Option.some(n)) // Some(84)

.getOrElse(0) // 84

```

### Either / Result – Fehlerbehandlung

```typescript

// Trägt entweder einen Fehler (Left) oder einen Erfolg (Right)

type Either<E, A> = Left<E> | Right<A>

  

parseInput(rawData) // Right(data) oder Left(ParseError)

.flatMap(validateData) // Right(data) oder Left(ValidationError)

.flatMap(saveToDb) // Right(result) oder Left(DbError)

.match({

left: err => console.error(err),

right: result => console.log(result)

})

```

### [[effect]] (Effect.ts) – Nebeneffekte

```typescript

// Effect<A, E, R> ist eine Monade, die Nebeneffekte, Fehler und Abhängigkeiten trägt

const program = Effect.gen(function* () {

const user = yield* fetchUser(id); // flatMap unter der Haube

const orders = yield* fetchOrders(user);

return orders;

});

```


> `Effect.gen` mit `yield*` ist syntaktischer Zucker für `.flatMap()` – genau wie `async/await` Zucker für `.then()` ist.


### Promise – asynchrone Berechnungen

```typescript

// Promise ist (fast) eine Monade

fetch('/api/user')

.then(res => res.json()) // .map()

.then(user => fetchOrders(user.id)) // .flatMap()

.catch(err => console.error(err))

```


> Promise ist nicht ganz eine Monade, weil sie eager (sofort ausführend) ist und `.then` beide Rollen (map + flatMap) übernimmt.

## Monade vs. Funktor

| | Funktor | Monade |

|---|---|---|

| Operation | `map(A → B)` | `flatMap(A → M<B>)` |

| Rückgabe | `M<B>` | `M<B>` (kein doppeltes Wrapping) |

| Beispiel | `Option.some(5).map(n => n * 2)` | `Option.some(5).flatMap(n => Option.some(n * 2))` |


Jede Monade ist auch ein Funktor – aber nicht umgekehrt.

## Warum Monaden?

- **Verkettung ohne if/else-Kaskaden**

- **Fehler und Sonderfälle werden durch den Typ ausgedrückt**, nicht durch Exceptions

- **Komposierbarkeit** – kleine Funktionen lassen sich zu größeren zusammensetzen

- Grundlage von Effect.ts, fp-ts, und funktionaler Programmierung allgemein

  

- [[effect]] – baut auf Monaden auf

- *"Monads for the Curious Programmer"* – (https://bartoszmilewski.com/2011/01/09/monads-for-the-curious-programmer-part-1/)
- https://bartoszmilewski.com/2011/03/14/monads-for-the-curious-programmer-part-2/




