
## Die Grundidee


| | CPU | GPU |

|---|---|---|

| Steht für | Central Processing Unit | Graphics Processing Unit |

| Kerne | Wenige, sehr mächtig (4–32) | Tausende, simpel (3000–16000+) |

| Stärke | Komplexe sequenzielle Logik | Massiv parallele Berechnungen |

| Typische Aufgaben | Businesslogik, I/O, OS | Grafik, ML, Simulationen |
  

## Wie die CPU denkt


Die CPU ist ein **Generalist** – sie kann alles, aber nacheinander. Ein Kern bearbeitet eine Aufgabe, dann die nächste.


```

CPU: [Task 1] → [Task 2] → [Task 3] → [Task 4]

sehr schnell pro Task, aber sequenziell

```


Gut für: Verzweigungen (`if/else`), komplexe Logik, kleine Datenmengen, I/O-Operationen.

## Wie die GPU denkt


Die GPU ist ein **Spezialist für Parallelität** – tausende einfache Kerne führen dieselbe Operation gleichzeitig auf riesigen Datensätzen aus.

  

```

GPU: [T1][T2][T3][T4][T5]...[T10000] ← alle gleichzeitig

langsamer pro Task, aber massiv parallel

```
  

Gut für: Matrixmultiplikation, Bildverarbeitung, neuronale Netze, Shader.
  

## Speicher: Wo landet was?


### RAM (Arbeitsspeicher) – CPU-Speicher


- Verbunden mit der CPU über den **System Bus**

- Typisch: 16–64 GB auf einem modernen Rechner

- Hier laufen: dein Node-Prozess, deine TypeScript-App, der Browser
  

```

Was im RAM liegt:

├── Laufende Prozesse (Node, Deno, Browser)

├── Variablen & Objekte deiner App

├── Geladene Dateien & Netzwerk-Responses

└── OS und laufende Programme

```

### VRAM (Video RAM) – GPU-Speicher

- Direkt auf der Grafikkarte, sehr schnelle Bandbreite

- Typisch: 8–24 GB (Consumer), 40–80 GB (ML-Karten wie A100, H100)

- **Komplett getrennt vom RAM** – Daten müssen explizit übertragen werden


```

Was im VRAM liegt:

├── Texturen & Framebuffer (Grafik)

├── ML-Modell-Gewichte (z.B. ein LLM)

├── Aktivierungen während des Forwards Pass

└── Gradients während des Trainings

```

### Der Flaschenhals: RAM ↔ VRAM Transfer
  

```

CPU/RAM ←──── PCIe Bus ────→ GPU/VRAM

langsam! (~16 GB/s)

(intern: RAM 50 GB/s, VRAM 900 GB/s)

```

Das Kopieren von Daten zwischen RAM und VRAM ist **teuer**. ML-Frameworks wie PyTorch minimieren das deshalb – einmal auf die GPU laden, dann so lange wie möglich dort lassen.


### Web & Backend (Node/Deno/TypeScript)

→ **Nur CPU & RAM relevant.** Die GPU ist für dich unsichtbar.


```typescript

// Das hier läuft alles auf CPU/RAM:

const users = await db.query("SELECT * FROM users");

const result = users.map(u => transform(u));

```

### Machine Learning / AI (Python, PyTorch, TensorFlow)

→ **GPU & VRAM werden zentral.**


```python

# Modell auf GPU laden

model = model.to("cuda") # von RAM → VRAM

  

# Daten auf GPU laden

tensor = tensor.to("cuda") # von RAM → VRAM

  

# Berechnung läuft auf GPU

output = model(tensor) # alles in VRAM

  

# Ergebnis zurück auf CPU

result = output.cpu().numpy() # von VRAM → RAM

```


### LLMs lokal ausführen (z.B. mit Ollama)

→ Das Modell muss **komplett in den VRAM passen**.


```

Llama 3 8B ≈ 16 GB VRAM (float16)

Llama 3 70B ≈ 140 GB VRAM (float16) → braucht mehrere GPUs

GPT-4 → läuft auf Hunderten GPUs in Rechenzentren

```


Wenn das Modell nicht in den VRAM passt, fällt es auf RAM zurück → deutlich langsamer (CPU-Inferenz).
  

### Shader / WebGL / WebGPU (Browser)

→ Du schreibst Code der **direkt auf der GPU läuft**.


```typescript

// WebGPU – GPU-Berechnungen direkt im Browser

const adapter = await navigator.gpu.requestAdapter();

const device = await adapter.requestDevice();

// Ab hier läuft Code auf der GPU

```

  
## Cache-Hierarchie (vollständiges Bild)


```

Schnellste, kleinste

↑

CPU Register (~1 ns, bytes)

L1 Cache (~1 ns, 64 KB pro Kern)

L2 Cache (~5 ns, 512 KB pro Kern)

L3 Cache (~20 ns, 16–64 MB, geteilt)

RAM (~100 ns, 16–64 GB)

SSD (~100 µs, 512 GB–4 TB)

HDD / Netzwerk (~10 ms+, unbegrenzt)

↓

Langsamste, größte

```


Die CPU-Caches (L1/L2/L3) sind vollautomatisch – als Entwicklerin hast du keinen direkten Einfluss darauf, aber **cache-freundlicher Code** (z.B. Arrays statt verkettete Listen) ist schneller weil er den L1/L2 Cache besser ausnutzt.


## Zusammenfassung
  

| Szenario | Relevanter Speicher |

|---|---|

| TypeScript / Node / Deno App | RAM |

| Datenbank-Queries | RAM + SSD |

| LLM lokal ausführen | VRAM (+ RAM als Fallback) |

| ML-Modell trainieren | VRAM |

| Browser-App mit 3D/Canvas | VRAM (via WebGL/WebGPU) |

| Alles andere im Web | RAM |