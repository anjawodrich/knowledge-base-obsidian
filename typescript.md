
TypeScript ist **JavaScript mit Typen**. Es ist eine Obermenge von JavaScript – jedes gültige JS ist auch gültiges TS. Der TypeScript Compiler (`tsc`) übersetzt TS zu JS und entfernt dabei alle Typen.

```
TypeScript = JavaScript + Typsystem + (manchmal neuere Syntax)
```

Typen existieren **nur zur Entwicklungszeit** – im Browser oder Node läuft am Ende reines JavaScript.

---

## Warum TypeScript?

```typescript
// JavaScript – Fehler erst zur Laufzeit sichtbar
function add(a, b) {
  return a + b;
}
add(1, "2"); // "12" – kein Fehler, falsches Ergebnis

// TypeScript – Fehler sofort im Editor sichtbar
function add(a: number, b: number): number {
  return a + b;
}
add(1, "2"); // ❌ Compilerfehler: Argument of type 'string' is not assignable to 'number'
```

---

## Grundlegende Typen

```typescript
// Primitive
const name: string = "Anja";
const age: number = 30;
const active: boolean = true;
const nothing: null = null;
const notDefined: undefined = undefined;

// Arrays
const numbers: number[] = [1, 2, 3];
const names: Array<string> = ["Anja", "Max"]; // gleichwertig

// Tuple – Array mit fixer Länge und Typen
const pair: [string, number] = ["Anja", 30];

// Any – deaktiviert das Typsystem ⚠️
const anything: any = "could be anything";

// Unknown – sicherer als any
const value: unknown = fetchSomething();
if (typeof value === "string") {
  value.toUpperCase(); // ✅ erst nach Type Guard nutzbar
}
```

---

## Interfaces & Types

Beide beschreiben die Form eines Objekts:

```typescript
// Interface
interface User {
  id: number;
  name: string;
  email?: string; // optional
}

// Type Alias
type User = {
  id: number;
  name: string;
  email?: string;
}
```

### Interface vs Type – wann was?

||Interface|Type|
|---|---|---|
|Objekte beschreiben|✅ bevorzugt|✅ möglich|
|Erweitern|`extends`|Intersection `&`|
|Union Types|❌|✅|
|Primitive benennen|❌|✅|
|Declaration Merging|✅ (automatisch)|❌|

```typescript
// Nur mit Type möglich:
type ID = string | number;  // Union
type Name = string;         // Primitiv benennen

// Interface erweitern:
interface Admin extends User {
  role: "admin";
}

// Type erweitern (Intersection):
type Admin = User & { role: "admin" };
```

**Faustregel:** Für Objekte und Klassen → `interface`. Für alles andere (Unions, Primitives, Utilities) → `type`.

---

## Union & Intersection Types

```typescript
// Union – entweder oder
type ID = string | number;
type Status = "active" | "inactive" | "pending"; // String Literal Union

// Intersection – beides gleichzeitig
type AdminUser = User & Admin;
```

---

## Generics – wiederverwendbare Typen

Generics erlauben es, Typen **parametrisierbar** zu machen – wie Funktionsparameter, aber für Typen.

```typescript
// Ohne Generic: nur für eine Typen nutzbar
function firstNumber(arr: number[]): number {
  return arr[0];
}

// Mit Generic: für jeden Typ nutzbar
function first<T>(arr: T[]): T {
  return arr[0];
}

first([1, 2, 3]);        // T wird zu number
first(["a", "b", "c"]); // T wird zu string

// Generic mit Einschränkung
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

getProperty({ name: "Anja", age: 30 }, "name"); // ✅
getProperty({ name: "Anja", age: 30 }, "foo");  // ❌ Compilerfehler
```

---

## Utility Types – eingebaute Typ-Helfer

TypeScript hat viele eingebaute Utility Types die häufige Transformationen erleichtern:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Partial – alle Felder optional
type UserUpdate = Partial<User>;
// { id?: number; name?: string; email?: string; password?: string }

// Required – alle Felder required
type StrictUser = Required<Partial<User>>;

// Pick – nur bestimmte Felder behalten
type PublicUser = Pick<User, "id" | "name">;
// { id: number; name: string }

// Omit – bestimmte Felder weglassen
type SafeUser = Omit<User, "password">;
// { id: number; name: string; email: string }

// Readonly – alle Felder unveränderlich
type FrozenUser = Readonly<User>;

// Record – Objekt mit definierten Keys und Wert-Typ
type StatusMap = Record<string, boolean>;
// { [key: string]: boolean }

// ReturnType – Rückgabetyp einer Funktion extrahieren
function fetchUser() { return { id: 1, name: "Anja" }; }
type FetchResult = ReturnType<typeof fetchUser>;
// { id: number; name: string }

// Parameters – Parameter-Typen einer Funktion
type FetchParams = Parameters<typeof fetchUser>;
```

---

## Type Guards – Typen zur Laufzeit einengen

```typescript
// typeof Guard
function format(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase(); // hier ist value: string
  }
  return value.toFixed(2);     // hier ist value: number
}

// instanceof Guard
function handle(error: Error | string) {
  if (error instanceof Error) {
    console.log(error.message);
  }
}

// Custom Type Guard (is)
interface Cat { meow(): void }
interface Dog { bark(): void }

function isCat(animal: Cat | Dog): animal is Cat {
  return "meow" in animal;
}
```

---

## Discriminated Unions – typsichere Zustände

Ein sehr mächtiges Pattern für Zustände die sich gegenseitig ausschließen:

```typescript
type LoadingState = { status: "loading" };
type SuccessState = { status: "success"; data: User[] };
type ErrorState   = { status: "error"; message: string };

type State = LoadingState | SuccessState | ErrorState;

function render(state: State) {
  switch (state.status) {
    case "loading": return "Laden...";
    case "success": return state.data; // TS weiß: data existiert hier
    case "error":   return state.message; // TS weiß: message existiert hier
  }
}
```

TypeScript prüft ob alle Fälle behandelt wurden – wenn du einen neuen Status hinzufügst und vergisst ihn im Switch zu behandeln, bekommst du einen Compilerfehler.

---

## `as const` – Werte einfrieren

```typescript
// Ohne as const: TS inferiert breite Typen
const directions = ["north", "south", "east", "west"];
// Typ: string[]

// Mit as const: TS inferiert enge Literal-Typen
const directions = ["north", "south", "east", "west"] as const;
// Typ: readonly ["north", "south", "east", "west"]

type Direction = typeof directions[number];
// "north" | "south" | "east" | "west"
```

**Use case:** Konstanten definieren ohne Enum – moderner, flexibler Ansatz.

---

## `satisfies` – Typ prüfen ohne einengen (seit TS 4.9)

```typescript
type Colors = "red" | "green" | "blue";
type Palette = Record<Colors, string>;

// satisfies prüft den Typ, lässt aber den inferrierten Typ stehen
const palette = {
  red: "#ff0000",
  green: "#00ff00",
  blue: "#0000ff",
} satisfies Palette;

// palette.red ist string (nicht "red" | "green" | "blue")
palette.red.toUpperCase(); // ✅ funktioniert
```

---

## tsconfig.json – wichtige Einstellungen

```json
{
  "compilerOptions": {
    "target": "ES2022",        // Welches JS wird ausgegeben
    "module": "ESNext",        // Modul-System
    "moduleResolution": "bundler", // Wie Imports aufgelöst werden
    "strict": true,            // Alle strikten Checks an ✅ immer einschalten
    "noUncheckedIndexedAccess": true, // arr[0] ist T | undefined, nicht T
    "exactOptionalPropertyTypes": true, // Unterscheidet undefined von fehlendem Feld
    "sourceMap": true,         // Für Debugging (TS-Zeilen im Stacktrace)
    "outDir": "./dist",        // Wohin der kompilierte JS-Code geht
    "rootDir": "./src"         // Wo dein TS-Code liegt
  }
}
```

**`strict: true`** aktiviert:

- `strictNullChecks` – `null` und `undefined` sind eigene Typen
- `noImplicitAny` – kein implizites `any`
- `strictFunctionTypes` – strengere Funktionstypen
- und weitere

---

## Häufige Fallen

### `any` vermeiden

```typescript
// ❌ deaktiviert das Typsystem komplett
const data: any = fetchData();

// ✅ unknown erzwingt Type Guard vor der Nutzung
const data: unknown = fetchData();
```

### Non-null Assertion `!` sparsam nutzen

```typescript
const el = document.getElementById("app")!; // sagst TypeScript "vertrau mir, ist nicht null"
// ⚠️ Wenn es doch null ist: Laufzeitfehler
```

### Type Assertion `as` ist kein Cast

```typescript
const value = fetchSomething() as string;
// TS glaubt dir – aber zur Laufzeit kann es trotzdem alles sein
```

---

## TypeScript & Effect.ts

Effect nutzt das TypeScript-Typsystem sehr tief – der Typ `Effect<A, E, R>` trägt Erfolg, Fehler und Abhängigkeiten direkt im Typ. Generics, Utility Types und Discriminated Unions sind dort alltäglich. Je besser du TypeScript verstehst, desto lesbarer wird Effect-Code.