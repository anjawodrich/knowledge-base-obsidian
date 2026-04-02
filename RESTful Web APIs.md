
## Was ist REST?
  

REST steht für **Representational State Transfer** – ein Architekturstil für Web-APIs, der 2000 von Roy Fielding definiert wurde.
  

Eine API ist "RESTful" wenn sie bestimmte Prinzipien befolgt. Sie ist kein Protokoll oder Standard, sondern ein **Designkonzept**.


## Die Kernidee: Ressourcen


In REST dreht sich alles um **Ressourcen** – Dinge, nicht Aktionen.
  

```

❌ Nicht REST: /getUser?id=1

❌ Nicht REST: /deleteUser?id=1

  

✅ REST: /users/1 (GET zum Lesen, DELETE zum Löschen)

```


Eine Ressource wird über eine **URL** identifiziert und mit **HTTP-Methoden** manipuliert.
  

## HTTP-Methoden & ihre Bedeutung
  

| Methode | Bedeutung | Idempotent? | Body? |

|--------|-----------|-------------|-------|

| `GET` | Ressource lesen | ✅ Ja | ❌ Nein |

| `POST` | Neue Ressource erstellen | ❌ Nein | ✅ Ja |

| `PUT` | Ressource vollständig ersetzen | ✅ Ja | ✅ Ja |

| `PATCH` | Ressource teilweise ändern | ❌ Nein* | ✅ Ja |

| `DELETE` | Ressource löschen | ✅ Ja | ❌ Nein |


> **Idempotent** = mehrfach ausführen hat denselben Effekt wie einmal ausführen.
  

## URL-Struktur
  

```

# Sammlung (Collection)

GET /users → alle User

POST /users → neuen User erstellen

  

# Einzelne Ressource

GET /users/42 → User 42 lesen

PUT /users/42 → User 42 ersetzen

PATCH /users/42 → User 42 teilweise ändern

DELETE /users/42 → User 42 löschen

  

# Verschachtelte Ressourcen

GET /users/42/orders → alle Orders von User 42

GET /users/42/orders/7 → Order 7 von User 42

```

  

## HTTP Status Codes


```

2xx – Erfolg

200 OK – Standard-Erfolg

201 Created – Ressource erstellt (nach POST)

204 No Content – Erfolg, kein Body (nach DELETE)

  

3xx – Weiterleitung

301 Moved Permanently

304 Not Modified – Cache ist aktuell

  

4xx – Client-Fehler (du hast etwas falsch gemacht)

400 Bad Request – Ungültige Anfrage

401 Unauthorized – Nicht eingeloggt

403 Forbidden – Eingeloggt, aber keine Berechtigung

404 Not Found – Ressource existiert nicht

409 Conflict – z.B. Email bereits vergeben

422 Unprocessable – Validierungsfehler

  

5xx – Server-Fehler (der Server hat etwas falsch gemacht)

500 Internal Server Error

503 Service Unavailable

```

  

## Request & Response
  

```http

# Request

POST /users HTTP/1.1

Content-Type: application/json

Authorization: Bearer <token>

  

{

"name": "Anja",

"email": "anja@example.com"

}

  

# Response

HTTP/1.1 201 Created

Content-Type: application/json

  

{

"id": 42,

"name": "Anja",

"email": "anja@example.com",

"createdAt": "2024-01-15T10:00:00Z"

}

```

  

## Die 6 REST-Prinzipien (vereinfacht)


1. **Client-Server** – Frontend und Backend sind getrennt

2. **Stateless** – Jeder Request enthält alle nötigen Infos (kein Session-State am Server)

3. **Cacheable** – Responses können gecacht werden

4. **Uniform Interface** – Einheitliche URLs und Methoden

5. **Layered System** – Client weiß nicht, ob er direkt mit dem Server oder einem Proxy spricht

6. **Code on Demand** (optional) – Server kann Code an den Client schicken


## REST vs. andere API-Stile
  

| | REST | GraphQL | RPC/tRPC |

|---|---|---|---|

| Struktur | Ressourcen-basiert | Query-basiert | Funktions-basiert |

| Overfetching | Kann passieren | Nein | Nein |

| Typsicherheit | Manuell | Schema | ✅ Automatisch |

| Lernkurve | Niedrig | Mittel | Niedrig (mit TS) |

| Gut für | Public APIs | Komplexe UIs | Interne TS-Projekte |