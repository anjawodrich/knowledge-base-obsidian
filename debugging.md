
## Grundkonzepte



## vs code: 
launch.json – Debugging konfigurieren

Unter `.vscode/launch.json` – hier definierst du wie VS Code dein Programm startet.

### Einzelnes TypeScript/Node File debuggen

```json

{

"version": "0.2.0",

"configurations": [

{

"type": "node",

"request": "launch",

"name": "Debug current file",

"program": "${file}",

"runtimeArgs": ["--loader", "ts-node/esm"],

"skipFiles": ["<node_internals>/**"]

}

]

}

```


> `${file}` = das aktuell geöffnete File in VS Code

### Mit ts-node (ohne separate Compilierung)


```json

{

"type": "node",

"request": "launch",

"name": "Run with ts-node",

"runtimeExecutable": "npx",

"runtimeArgs": ["ts-node", "${file}"],

"skipFiles": ["<node_internals>/**"]

}

```

  
### Attach to running process (z.B. laufende Node-App)


```bash

# Node-Prozess mit Debug-Port starten

node --inspect src/index.ts

```


```json

{

"type": "node",

"request": "attach",

"name": "Attach to process",

"port": 9229

}

```


## Breakpoints setzen

  

- **Klick links neben die Zeilennummer** → roter Punkt = Breakpoint

- **Rechtsklick auf Breakpoint** → Conditional Breakpoint (nur pausieren wenn Bedingung zutrifft)

- **Logpoint** (Rechtsklick) → `console.log` ohne Code zu ändern, nur im Debug-Mode


```javascript

// Conditional Breakpoint Beispiel:

// Nur pausieren wenn user.id === 42

user.id === 42

```

  

## Keyboard Shortcuts (Mac)

  

| Aktion | Shortcut |

|--------|----------|

| Debugging starten | `F5` |

| Debugging stoppen | `Shift + F5` |

| Step Over | `F10` |

| Step Into | `F11` |

| Step Out | `Shift + F11` |

| Continue (bis nächster Breakpoint) | `F5` |

| Alle Breakpoints toggle | `Cmd + Shift + F9` |

  

## Debug Console

  

Wenn das Programm an einem Breakpoint pausiert:

- **Debug Console** öffnet sich unten

- Du kannst dort direkt Ausdrücke evaluieren:

```

> user.name

> orders.length

> JSON.stringify(response, null, 2)

```

## Nützliche Features

### Watch Expressions

Im Debug-Panel unter "Watch" → `+` → Ausdruck eingeben

Wird bei jedem Schritt neu ausgewertet.

### Inline Values

VS Code zeigt während des Debuggens Variablenwerte direkt im Code an (graue Schrift rechts von der Zeile).

### Restart Frame

Im Call Stack → Rechtsklick auf einen Frame → "Restart Frame"

Springt zurück zum Anfang dieser Funktion, ohne das Programm neu zu starten.

## Häufige Probleme
  

**Breakpoint wird nicht getroffen:**

- Source Maps fehlen → in `tsconfig.json` prüfen: `"sourceMap": true`

- Falsches File wird ausgeführt (compiled JS statt TS)


**`cannot find module` beim Debuggen:**

- `cwd` in launch.json setzen: `"cwd": "${workspaceFolder}"`


**Debugger attached aber kein Halt:**

- `skipFiles` zu aggressiv → Node-interne Files können übersprungen werden, eigene nicht