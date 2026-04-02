
Beide sind **Package Manager für Node.js** – sie installieren Abhängigkeiten, verwalten `package.json` und lösen Versionen auf. Der Unterschied liegt im **wie**.

## Das Problem mit npm: Disk-Duplikation

```
projekt-a/node_modules/react  (v18)
projekt-b/node_modules/react  (v18)
projekt-c/node_modules/react  (v18)
→ dieselbe Library 3x auf der Festplatte
```

npm kopiert jedes Package in jeden `node_modules`-Ordner – auch wenn es bereits woanders installiert ist.

## pnpm löst das mit einem Content-Addressable Store

```
~/.pnpm-store/
  react@18.0.0/   ← einmal gespeichert

projekt-a/node_modules/react  → Symlink zum Store
projekt-b/node_modules/react  → Symlink zum Store
projekt-c/node_modules/react  → Symlink zum Store
```

React liegt **einmal auf der Festplatte**, alle Projekte verlinken darauf. Das spart massiv Speicherplatz und macht Installationen schneller.

## Vergleich

**Speicherplatz**
**npm**: Viel (Duplikate)
**pnpm**: Wenig (zentraler Store)

**Installationsgeschwindigkeit**
**npm**: Mittel
**pnpm**: Schneller (Cache-Hits)

**`node_modules` Struktur**
**npm**: Flach (hoisting)
**pnpm**: Strikt (kein Phantom-Access)

**Monorepo-Support**
**npm**: Workspaces (basic)
**pnpm**: Workspaces (sehr gut)

## Phantom Dependencies – das stille npm-Problem

npm "hoistet" Packages nach oben – dadurch kannst du Packages importieren, die **nicht** in deiner `package.json` stehen:

```typescript
// Du hast lodash nicht in package.json
// Aber ein anderes Package hat es installiert
// npm macht es trotzdem verfügbar → funktioniert zufällig
import _ from 'lodash'; // ✅ mit npm (zufällig)
                        // ❌ mit pnpm (korrekt geblockt)
```

pnpm erlaubt nur den Zugriff auf explizit deklarierte Abhängigkeiten. Das macht den Code robuster.

## Befehle im Vergleich

```bash
# Installieren
npm install          →  pnpm install
npm install react    →  pnpm add react
npm install -D vitest →  pnpm add -D vitest

# Ausführen
npm run dev          →  pnpm dev  (oder pnpm run dev)
npx tsc              →  pnpm dlx tsc  (oder pnpm exec tsc)

# Entfernen
npm uninstall react  →  pnpm remove react
```