## Was bedeutet "kompilieren"?

Kompilieren bedeutet: **Quellcode in eine andere Form umwandeln**, die ausgeführt werden kann.

Bei klassischen Sprachen wie C oder Java:

```
Quellcode (.c / .java)
       ↓ Compiler
Maschinencode / Bytecode
       ↓
Ausführbar
```

Bei JavaScript ist es komplizierter – weil JS ursprünglich **interpretiert** wurde, nicht kompiliert.

---

## JavaScript: Interpretiert oder kompiliert?

**Ursprünglich:** JavaScript war eine interpretierte Sprache – der Browser las den Code Zeile für Zeile und führte ihn direkt aus. Kein Zwischenschritt.

**Heute:** Moderne JavaScript-Engines (V8 in Chrome/Node, SpiderMonkey in Firefox) nutzen **JIT-Kompilierung** (Just-In-Time). Das bedeutet:

```
JS-Quellcode
     ↓
Parsing → AST (Abstract Syntax Tree)
     ↓
Bytecode (schnell, grob)
     ↓ (wenn Code oft ausgeführt wird)
Optimierter Maschinencode (JIT)
```

Der Code wird also **während der Ausführung** kompiliert – nicht vorher. Das ist warum JavaScript über die Jahre so viel schneller geworden ist.

---

## TypeScript kompilieren

TypeScript ist kein JavaScript – Browser und Node verstehen es **nicht direkt**. Es muss erst in JavaScript umgewandelt werden:

```
TypeScript (.ts)
      ↓ tsc (TypeScript Compiler)
JavaScript (.js)
      ↓
Browser / Node führt es aus
```

Das ist gemeint wenn jemand sagt "TypeScript kompilieren" – es bedeutet: TS → JS übersetzen. Typen werden dabei **komplett entfernt**, sie existieren nur zur Entwicklungszeit.

```typescript
// TypeScript (vor dem Kompilieren)
function greet(name: string): string {
  return `Hello ${name}`;
}

// JavaScript (nach dem Kompilieren)
function greet(name) {
  return `Hello ${name}`;
}
```

### Tools die TypeScript kompilieren

|Tool|Beschreibung|
|---|---|
|`tsc`|Offizieller TypeScript Compiler, langsam aber exakt|
|`esbuild`|Extrem schnell, kein Type-Checking|
|`swc`|Schnell, in Rust geschrieben, kein Type-Checking|
|`ts-node`|Führt TS direkt aus (kompiliert on-the-fly)|
|`tsx`|Wie ts-node, aber schneller|
|Deno|Kompiliert TS nativ, kein Setup nötig|

> `esbuild` und `swc` kompilieren TS zu JS, prüfen aber **keine Typen** – das macht nur `tsc`. In der Praxis: `tsc` für Type-Checking, `esbuild`/`swc` für schnelle Builds.

---

## Was ist die Runtime?

Die **Runtime** (Laufzeitumgebung) ist die Umgebung, in der JavaScript tatsächlich ausgeführt wird. Sie stellt bereit:

- Die **JS-Engine** (führt den Code aus)
- **APIs** die nicht Teil von JavaScript selbst sind (z.B. `fetch`, `setTimeout`, `fs`)
- Den **Event Loop**

```
Dein JS-Code
     +
Runtime (Engine + APIs + Event Loop)
     =
Ausführbares Programm
```

### Die wichtigsten Runtimes

|Runtime|Engine|Umgebung|Besonderheit|
|---|---|---|---|
|**Browser**|V8 (Chrome), SpiderMonkey (Firefox)|Frontend|DOM, window, localStorage|
|**Node.js**|V8|Backend/CLI|fs, http, process|
|**Deno**|V8|Backend/CLI|Permissions, TS nativ|
|**Bun**|JavaScriptCore (Safari)|Backend/CLI|Sehr schnell, all-in-one|

### Warum Runtime wichtig ist

Dieselbe Sprache, aber unterschiedliche APIs:

```typescript
// Nur im Browser verfügbar:
document.getElementById("app");
localStorage.setItem("key", "value");
window.location.href;

// Nur in Node verfügbar:
import fs from "fs";
fs.readFileSync("./file.txt");
process.env.DATABASE_URL;

// In beiden verfügbar (moderne Runtimes):
fetch("https://api.example.com");
setTimeout(() => {}, 1000);
console.log();
```

---

## Der Event Loop – wie JavaScript async funktioniert

JavaScript ist **single-threaded** – es kann nur eine Sache gleichzeitig tun. Trotzdem ist es nicht blockierend. Das ermöglicht der **Event Loop**.

```
┌─────────────────────────┐
│        Call Stack        │  ← Hier läuft dein Code (synchron)
└─────────────────────────┘
           ↑
           │ wenn leer
           │
┌─────────────────────────┐
│      Callback Queue      │  ← Fertige async-Callbacks warten hier
│  (Microtask Queue zuerst)│
└─────────────────────────┘
           ↑
           │
┌─────────────────────────┐
│      Web APIs / I/O      │  ← setTimeout, fetch, fs laufen hier
└─────────────────────────┘
```

```typescript
console.log("1");           // Call Stack

setTimeout(() => {
  console.log("2");         // Callback Queue (später)
}, 0);

Promise.resolve().then(() => {
  console.log("3");         // Microtask Queue (vor setTimeout!)
});

console.log("4");           // Call Stack

// Ausgabe: 1, 4, 3, 2
```

**Microtask Queue** (Promises) hat **Vorrang** vor der normalen Callback Queue (setTimeout, I/O).

---

## Bundling – was ist ein Bundler?

Im Browser kannst du nicht einfach `import` aus 100 verschiedenen Files nutzen – das wäre zu langsam (100 HTTP-Requests). Ein **Bundler** fasst alles zu einer oder wenigen Dateien zusammen:

```
src/
  index.ts
  utils.ts
  components/Button.ts
  node_modules/react/...
        ↓ Bundler (Vite, esbuild, Webpack)
dist/
  bundle.js   ← alles in einer Datei
```

### Gängige Bundler

|Tool|Stärke|
|---|---|
|**Vite**|Schnell in Dev, moderner Standard|
|**esbuild**|Extrem schnell, wird oft intern genutzt|
|**Webpack**|Alt, komplex, aber sehr flexibel|
|**Rollup**|Gut für Libraries|

---

## Zusammenfassung: Der Weg von TypeScript zur Ausführung

```
Du schreibst:     TypeScript (.ts)
                       ↓
Compiler:         tsc / esbuild / swc
                  → TypeScript → JavaScript
                  → Typen werden entfernt
                       ↓
Optional:         Bundler (Vite, esbuild)
                  → viele JS-Files → ein Bundle
                       ↓
Runtime:          Node / Deno / Browser
                  → JIT-Kompilierung (V8)
                  → Event Loop
                  → Ausführung
```

---

## Begriffe auf einen Blick

|Begriff|Bedeutung|
|---|---|
|**Kompilieren**|Quellcode in eine andere Form umwandeln|
|**Transpilieren**|Von einer Hochsprache in eine andere (TS → JS)|
|**Runtime**|Umgebung die den Code ausführt|
|**Engine**|Der eigentliche JS-Interpreter/Compiler (V8 etc.)|
|**JIT**|Just-In-Time: Kompilierung während der Ausführung|
|**Bundler**|Fasst viele Files zu einem zusammen|
|**Event Loop**|Ermöglicht async in single-threaded JS|
|**Call Stack**|Wo synchroner Code ausgeführt wird|
|**Microtask Queue**|Warteschlange für Promises|
