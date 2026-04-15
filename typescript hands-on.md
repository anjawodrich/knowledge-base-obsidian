# TypeScript – Verschachtelte Typen

## Grundprinzip: Typen die andere Typen berechnen

In TypeScript können Typen nicht nur beschreiben **was** ein Wert ist, sondern auch **berechnet** werden – basierend auf anderen Typen. Das nennt man **Type-Level Programming**.

```typescript
// Wert-Level: eine Funktion die Werte transformiert
const toUpper = (s: string) => s.toUpperCase();

// Type-Level: ein Typ der andere Typen transformiert
type ToArray<T> = T[];
type Names = ToArray<string>; // string[]
```

---

## Beispiel 1 erklärt

```typescript
<const Template extends string>(template: Template) => Schema.Schema<Params<Template>, string, never>
```

Zerlegt:

```typescript
<const Template extends string>
// Generic-Parameter namens Template
// extends string → Template muss ein String sein
// const → TS soll den engsten Typ inferieren (Literal, nicht string)
//         z.B. "/users/:id" statt einfach string

(template: Template)
// Funktionsparameter: template hat denselben Typ wie Template
// Das erlaubt TS, den exakten Wert im Typ zu "merken"

=> Schema.Schema<Params<Template>, string, never>
// Rückgabetyp: Schema.Schema mit drei Typ-Parametern:
//   1. Params<Template>  → berechnet aus dem Template-Wert
//   2. string            → Input-Typ
//   3. never             → keine Abhängigkeiten

// Params<Template> ist selbst ein Typ der den String analysiert:
// Params<"/users/:id/:name"> → { id: string; name: string }
```

**Was das tut in der Praxis:**

```typescript
const schema = makeSchema("/users/:id/:name");
//    schema hat Typ: Schema.Schema<{ id: string; name: string }, string, never>
//    TypeScript hat den Pfad-String analysiert und daraus einen Objekt-Typ gebaut!
```

---

## Mapped Types – die Grundlage von Beispiel 2

Bevor Beispiel 2:

```typescript
// Mapped Type: iteriert über eine Union und erstellt Objekt-Felder
type Flags = { [K in "a" | "b" | "c"]: boolean };
// { a: boolean; b: boolean; c: boolean }

// Mit generischem Typ
type Optional<T> = { [K in keyof T]?: T[K] };
// Macht alle Felder von T optional
```

---

## Beispiel 2 erklärt

```typescript
{ [Param in ExtractParamNames<Template, []>]: typeof Schema.String; }
```

Zerlegt:

```typescript
ExtractParamNames<Template, []>
// Ein Typ der alle Parameter-Namen aus dem Template extrahiert
// z.B. ExtractParamNames<"/users/:id/:name", []>
//    → "id" | "name"   (eine String-Literal-Union)

[Param in ExtractParamNames<Template, []>]
// Mapped Type: iteriert über jedes Element der Union
// Param nimmt nacheinander jeden Wert an: "id", dann "name"

typeof Schema.String
// typeof holt den Typ eines Wertes (nicht einer Typdeklaration)
// Schema.String ist ein Wert → typeof gibt seinen Typ zurück

// Zusammen:
{ [Param in "id" | "name"]: typeof Schema.String }
// ergibt:
{ id: typeof Schema.String; name: typeof Schema.String }
```

**Schritt für Schritt visualisiert:**

```typescript
// Template = "/users/:id/:name"

// Schritt 1: ExtractParamNames extrahiert
ExtractParamNames<"/users/:id/:name", []>
// → "id" | "name"

// Schritt 2: Mapped Type iteriert
{ [Param in "id" | "name"]: typeof Schema.String }
// → { id: typeof Schema.String; name: typeof Schema.String }

// Das ist das Schema das die Parameter validiert
```

---

## Wie ExtractParamNames funktioniert (Konzept)

Solche Typen nutzen **Template Literal Types** und **Rekursion**:

```typescript
// Vereinfachtes Beispiel
type ExtractParamNames<
  S extends string,
  Acc extends string[] = []
> =
  S extends `:${infer Param}/${infer Rest}`
    ? ExtractParamNames<Rest, [...Acc, Param]>  // Rekursion
    : S extends `:${infer Param}`
    ? [...Acc, Param][number]                   // letzter Parameter
    : Acc[number];                              // fertig, gib Union zurück

// Ablauf für "/users/:id/:name":
// → S = "users/:id/:name"
// → matcht `:${infer Param}/${infer Rest}` → Param="id", Rest="name"
// → Rekursion mit Rest="name", Acc=["id"]
// → matcht `:${infer Param}` → Param="name"
// → gibt ["id", "name"][number] zurück = "id" | "name"
```

**`infer`** bedeutet: "Leite diesen Typ ab und gib ihm einen Namen."

---

## Beispiel 3 erklärt – Funktions-Overloads

```typescript
const path: {
  <Template extends `/${string}`>(template: Template): Paseo.Paseo<{
    path: Path.Params<Template>;
  }, Response.Response<null, { readonly status: 404; }>, Request.Request | Scope.Scope, Path.Meta<Template>>;

  <Template extends `/${string}`, E, R>(template: Template, handleError: (error: ParseError) => Effect.Effect<never, E, R>): Paseo.Paseo<{
    path: Path.Params<Template>;
  }, E, Request.Request | Scope.Scope | R, Path.Meta<Template>>;
}
```

Das ist ein **Overload-Typ** – `path` kann auf **zwei Arten** aufgerufen werden:

### Overload 1 – ohne Error Handler

```typescript
<Template extends `/${string}`>(template: Template):
  Paseo.Paseo<
    { path: Path.Params<Template> },           // Context: extrahierte Pfad-Parameter
    Response.Response<null, { readonly status: 404 }>, // Error: immer 404
    Request.Request | Scope.Scope,             // Requirements
    Path.Meta<Template>                        // Metadata über den Pfad
  >
```

```typescript
// Verwendung:
const route = path("/users/:id");
// Fehler → automatisch 404
```

### Overload 2 – mit eigenem Error Handler

```typescript
<Template extends `/${string}`, E, R>(
  template: Template,
  handleError: (error: ParseError) => Effect.Effect<never, E, R>
):
  Paseo.Paseo<
    { path: Path.Params<Template> },
    E,                                         // Error-Typ kommt vom Handler
    Request.Request | Scope.Scope | R,         // Requirements + Handler-Requirements
    Path.Meta<Template>
  >
```

```typescript
// Verwendung:
const route = path("/users/:id", (error) =>
  Effect.fail(new CustomError(error))
);
// Fehler → CustomError (eigener Typ E)
```

### Warum Overloads?

```typescript
// TypeScript wählt automatisch den richtigen Overload:
path("/users/:id")
// → Overload 1: Error ist Response.Response<null, { status: 404 }>

path("/users/:id", handleError)
// → Overload 2: Error ist whatever handleError zurückgibt (E)
```

---

## `readonly` in Typen

```typescript
{ readonly status: 404 }
// readonly: das Feld kann nach der Erstellung nicht mehr geändert werden
// 404: Literal-Typ – nicht irgendeine number, exakt 404

// Vergleich:
type A = { status: number }  // status kann jede Zahl sein
type B = { status: 404 }     // status kann nur 404 sein
type C = { readonly status: 404 } // status ist 404 und unveränderlich
```

---

## Union in Requirements: `Request.Request | Scope.Scope | R`

```typescript
Request.Request | Scope.Scope | R
// Das sind die "Requirements" – was der Effect braucht um zu laufen
// | bedeutet: alle diese Dinge müssen bereitgestellt werden
// R kommt vom handleError-Handler – seine Requirements werden hier hinzugefügt
```

Das ist das **Effect-Dependency-System** im Typ: wenn dein Handler zusätzliche Services braucht, tauchen sie automatisch im Rückgabe-Typ auf.

---

## Muster-Übersicht

|Muster|Syntax|Bedeutung|
|---|---|---|
|Generic|`<T>`|Typ-Parameter|
|Constraint|`T extends string`|T muss string sein|
|Const Generic|`const T extends string`|engster Literal-Typ|
|Mapped Type|`{ [K in Union]: Type }`|Objekt aus Union bauen|
|Conditional Type|`T extends X ? A : B`|Typ-Verzweigung|
|Infer|`T extends ${infer P}`|Teiltyp ableiten|
|Template Literal|`` `/${string}` ``|String-Muster als Typ|
|Readonly|`{ readonly x: T }`|unveränderliches Feld|
|Literal Type|`404`, `"get"`|exakter Wert als Typ|
|Union|`A \| B`|entweder A oder B|
|Intersection|`A & B`|A und B gleichzeitig|
|typeof|`typeof value`|Typ eines Wertes|
|keyof|`keyof T`|Union aller Keys von T|
|Overload|`{ (a): X; (a, b): Y }`|verschiedene Signaturen|
