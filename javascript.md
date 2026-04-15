## Arrays

### `.map()` – Transformieren

Erstellt ein **neues Array** indem jedes Element transformiert wird. Das Original bleibt unverändert.

```typescript
const prices = [10, 20, 30];
const withTax = prices.map(p => p * 1.2); // [12, 24, 36]

const users = [{ name: "Anja" }, { name: "Max" }];
const names = users.map(u => u.name); // ["Anja", "Max"]
```

**Use case:** Daten umformen, z.B. API-Response in UI-Format konvertieren.  
**⚠️ Nachteil:** Erstellt immer ein neues Array – bei sehr großen Arrays (100k+ Elemente) ist eine `for`-Schleife schneller weil kein neues Array allokiert wird.

---

### `.filter()` – Filtern

Gibt ein **neues Array** zurück mit nur den Elementen, die die Bedingung erfüllen.

```typescript
const numbers = [1, 2, 3, 4, 5];
const even = numbers.filter(n => n % 2 === 0); // [2, 4]

const orders = [...];
const shipped = orders.filter(o => o.status === "shipped");
```

**Use case:** Listen eingrenzen, z.B. nur aktive User, nur offene Tickets.  
**⚠️ Nachteil:** Durchläuft immer das gesamte Array. Bei großen Datensätzen besser in der Datenbank filtern statt im JavaScript.

---

### `.reduce()` – Aggregieren

Reduziert ein Array auf **einen einzelnen Wert** – der flexibelste Array-Operator.

```typescript
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((acc, n) => acc + n, 0); // 10

// Objekt aus Array bauen
const users = [{ id: 1, name: "Anja" }, { id: 2, name: "Max" }];
const byId = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});
// { 1: { id: 1, name: "Anja" }, 2: { id: 2, name: "Max" } }
```

**Use case:** Summen, Gruppierungen, Arrays in Objekte umwandeln.  
**⚠️ Nachteil:** Schwer lesbar wenn komplex. Für einfache Summen lieber `map` + `filter` kombinieren – lesbarer. `reduce` nur wenn wirklich nötig.

---

### `.find()` vs `.filter()` – Einzelnes Element suchen

```typescript
const users = [{ id: 1 }, { id: 2 }, { id: 3 }];

users.find(u => u.id === 2);    // { id: 2 } – stoppt beim ersten Treffer ✅
users.filter(u => u.id === 2);  // [{ id: 2 }] – durchläuft alles ❌
```

**Use case:** Einzelnes Element nach ID oder Bedingung suchen.  
**✅ Performance:** `find` ist schneller als `filter` wenn du nur ein Element brauchst – es bricht beim ersten Treffer ab.

---

### `.findIndex()` – Index eines Elements

```typescript
const items = ["a", "b", "c"];
const idx = items.findIndex(i => i === "b"); // 1
```

**Use case:** Position eines Elements finden, z.B. um es danach zu ersetzen oder zu löschen.

---

### `.some()` vs `.every()` – Bedingung prüfen

```typescript
const numbers = [1, 2, 3, 4];

numbers.some(n => n > 3);   // true  – mindestens eines erfüllt die Bedingung
numbers.every(n => n > 0);  // true  – alle erfüllen die Bedingung
numbers.every(n => n > 2);  // false
```

**✅ Performance:** Beide brechen früh ab – `some` beim ersten `true`, `every` beim ersten `false`. Besser als `.filter().length > 0`.

---

### `.flat()` und `.flatMap()` – Verschachtelte Arrays

```typescript
const nested = [[1, 2], [3, 4], [5]];
nested.flat(); // [1, 2, 3, 4, 5]

// flatMap = map + flat in einem Schritt
const sentences = ["hello world", "foo bar"];
sentences.flatMap(s => s.split(" ")); // ["hello", "world", "foo", "bar"]
```

**Use case:** API gibt Arrays von Arrays zurück, oder du erzeugst mit `map` verschachtelte Arrays.

---

### `.includes()` vs `Set.has()` – Enthält-Prüfung

```typescript
// Array.includes – O(n), durchläuft das Array
const arr = [1, 2, 3, 4, 5];
arr.includes(3); // true

// Set.has – O(1), sofort
const set = new Set([1, 2, 3, 4, 5]);
set.has(3); // true
```

**✅ Performance:** Bei häufigen Enthält-Prüfungen auf großen Listen immer `Set` verwenden. `includes` auf Arrays ist O(n) – bei 100k Elementen sehr langsam.

---

### `.sort()` – Sortieren

```typescript
// ⚠️ Falle: Ohne Callback sortiert JS nach Unicode (Strings!)
[10, 2, 1, 20].sort();           // [1, 10, 2, 20] ← FALSCH
[10, 2, 1, 20].sort((a, b) => a - b); // [1, 2, 10, 20] ← richtig

// Objekte sortieren
users.sort((a, b) => a.name.localeCompare(b.name));
```

**⚠️ Nachteil:** `.sort()` **mutiert** das Original-Array! Wenn du das nicht willst:

```typescript
const sorted = [...arr].sort((a, b) => a - b); // Kopie erst
```

---

### `Array.from()` und Spread `[...]` – Array erstellen

```typescript
// Aus Set (Duplikate entfernen)
const unique = Array.from(new Set([1, 1, 2, 3, 3])); // [1, 2, 3]
// oder:
const unique2 = [...new Set([1, 1, 2, 3, 3])];

// Array mit fixer Länge erstellen
Array.from({ length: 5 }, (_, i) => i); // [0, 1, 2, 3, 4]
```

---

## Objekte

### `Object.keys()`, `Object.values()`, `Object.entries()`

```typescript
const user = { name: "Anja", age: 30, city: "Wien" };

Object.keys(user);    // ["name", "age", "city"]
Object.values(user);  // ["Anja", 30, "Wien"]
Object.entries(user); // [["name", "Anja"], ["age", 30], ["city", "Wien"]]

// Häufiges Muster: Objekt transformieren
const upper = Object.fromEntries(
  Object.entries(user).map(([k, v]) => [k, String(v).toUpperCase()])
);
```

---

### Spread `{...}` vs `Object.assign()` – Objekte zusammenführen

```typescript
const base = { a: 1, b: 2 };
const extra = { b: 99, c: 3 };

// Spread (modern, bevorzugt)
const merged = { ...base, ...extra }; // { a: 1, b: 99, c: 3 }

// Object.assign (mutiert das erste Objekt!)
Object.assign(base, extra); // base ist jetzt { a: 1, b: 99, c: 3 } ⚠️
```

**⚠️ Wichtig:** Beide machen nur einen **shallow copy** – verschachtelte Objekte werden nicht tief kopiert:

```typescript
const obj = { a: { b: 1 } };
const copy = { ...obj };
copy.a.b = 99;
console.log(obj.a.b); // 99 ← Original wurde mitverändert!

// Tiefer Clone:
const deepCopy = structuredClone(obj); // modern, eingebaut seit Node 17
```

---

### `structuredClone()` – Tiefer Clone

```typescript
const original = { user: { name: "Anja", scores: [1, 2, 3] } };
const clone = structuredClone(original);
clone.user.name = "Max";
console.log(original.user.name); // "Anja" ← unverändert ✅
```

**Use case:** Objekt kopieren ohne Referenz-Probleme.  
**⚠️ Nachteil:** Funktioniert nicht mit Funktionen, `undefined`, `Symbol`, oder Klassen-Instanzen – nur mit plain data.

---

## Strings

### Template Literals

```typescript
const name = "Anja";
const greeting = `Hallo ${name}, du bist ${2025 - 1990} Jahre alt.`;
```

---

### `.split()` / `.join()`

```typescript
"hello world foo".split(" ");     // ["hello", "world", "foo"]
["hello", "world"].join(", ");    // "hello, world"
```

---

### `.trim()`, `.trimStart()`, `.trimEnd()`

```typescript
"  hello  ".trim();       // "hello"
"  hello  ".trimStart();  // "hello  "
```

**Use case:** User-Input bereinigen bevor Validierung.

---

### `.includes()`, `.startsWith()`, `.endsWith()`

```typescript
"hello world".includes("world");     // true
"hello world".startsWith("hello");   // true
"hello world".endsWith("world");     // true
```

---

### `.replaceAll()`

```typescript
"a-b-c-d".replaceAll("-", "/"); // "a/b/c/d"
// .replace() ersetzt nur das erste Vorkommen!
```

---

## Asynchron

### `Promise.all()` – Parallel ausführen

```typescript
// Sequenziell: langsam (wartet auf jedes einzeln)
const user = await fetchUser(1);
const orders = await fetchOrders(1);

// Parallel: schnell (läuft gleichzeitig)
const [user, orders] = await Promise.all([fetchUser(1), fetchOrders(1)]);
```

**✅ Performance:** Wenn Requests voneinander unabhängig sind, immer `Promise.all` statt `await` in Reihe.  
**⚠️ Nachteil:** Wenn ein Promise fehlschlägt, schlägt alles fehl.

---

### `Promise.allSettled()` – Parallel, Fehler tolerant

```typescript
const results = await Promise.allSettled([fetchA(), fetchB(), fetchC()]);

results.forEach(r => {
  if (r.status === "fulfilled") console.log(r.value);
  if (r.status === "rejected") console.error(r.reason);
});
```

**Use case:** Mehrere unabhängige Requests, bei denen einzelne Fehler OK sind.

---

### `Promise.race()` – Schnellster gewinnt

```typescript
const result = await Promise.race([
  fetch("/api/fast"),
  new Promise((_, reject) => setTimeout(() => reject("timeout"), 3000))
]);
```

**Use case:** Timeout implementieren, oder mehrere redundante Quellen anfragen.

---

## Nützliche Utilities

### Nullish Coalescing `??` vs `||`

```typescript
// || gibt rechts zurück bei ALLEN falsy Werten (0, "", false, null, undefined)
const count = userCount || 10;   // ⚠️ wenn userCount = 0, gibt es 10 zurück!

// ?? gibt rechts nur bei null oder undefined zurück
const count = userCount ?? 10;   // ✅ wenn userCount = 0, gibt es 0 zurück
```

---

### Optional Chaining `?.`

```typescript
const city = user?.address?.city;
// Ohne: user && user.address && user.address.city

// Mit Methoden
const lower = str?.toLowerCase();

// Mit Arrays
const first = arr?.[0];
```

---

### `Object.freeze()` – Immutabilität erzwingen

```typescript
const config = Object.freeze({ apiUrl: "https://api.example.com" });
config.apiUrl = "something else"; // Silently ignored (strict mode: Error)
```

**⚠️ Nachteil:** Auch nur shallow – verschachtelte Objekte sind nicht gefroren.

---

## Performance-Übersicht

|Operation|Schnell ✅|Langsam ⚠️|
|---|---|---|
|Element suchen|`Set.has()`, `Map.get()`, `find()`|`includes()` auf großem Array|
|Existenz prüfen|`some()`|`filter().length > 0`|
|Objekt klonen (flach)|Spread `{...}`|`JSON.parse(JSON.stringify())`|
|Objekt klonen (tief)|`structuredClone()`|`JSON.parse(JSON.stringify())`|
|Parallele Requests|`Promise.all()`|`await` in Reihe|
|Großes Array iterieren|`for`-Schleife|`.map()` / `.filter()`|
|String zusammenbauen|Template Literals|`+` Konkatenation in Schleife|
