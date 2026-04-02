

Ubiquitous Language (dt. *allgegenwärtige Sprache*) ist ein Konzept aus [[domain driven design]] (DDD) von Eric Evans.

Die Idee: Entwickler und Fachexperten (Product Owner, Stakeholder, Domänen-Experten) sprechen **dieselbe Sprache** – und diese Sprache spiegelt sich direkt im Code wider und wird in demselben, also im [[bounded context]] verwendet.
  
## Das Problem ohne Ubiquitous Language


```

Fachexpertin sagt: "Eine Bestellung wird aufgegeben und dann versandt."

Entwickler nennt: OrderEntity → createOrder() → shipItem()

  

→ "Bestellung aufgeben" ≠ createOrder()

→ "versandt" ≠ shipItem()

→ Übersetzungsfehler entstehen → Bugs entstehen

```

## Das Ziel

```

Fachexpertin sagt: "Eine Bestellung wird aufgegeben und dann versandt."

Code sagt: Order → placeOrder() → dispatch()

  

→ Die Sprache ist dieselbe in: Meetings, Dokumentation, Code, Datenbank

```

  

## Praktisches Beispiel

  

**Ohne Ubiquitous Language:**

```typescript

// Was ist ein "user" hier? Kunde? Admin? Beides?

// Was bedeutet "process"?

function processUser(user: UserEntity) {

if (user.type === 1) {

sendEmail(user);

updateRecord(user);

}

}

```

  

**Mit Ubiquitous Language:**

```typescript

// Klare Begriffe aus der Domäne

function onboardCustomer(customer: Customer) {

sendWelcomeEmail(customer);

activateAccount(customer);

}

```

  

## Wie man Ubiquitous Language aufbaut

  

1. **Event Storming / Domain Workshops** – Entwickler und Experten modellieren gemeinsam

2. **Glossar anlegen** – gemeinsames Dokument mit definierten Begriffen

3. **Begriffe im Code durchziehen** – Klassen, Methoden, Variablen, DB-Spalten

4. **Konsequent korrigieren** – wenn jemand einen anderen Begriff verwendet, sofort klären

  

## Typische Begriffe die definiert werden sollten

  

- **Entitäten** (z.B. `Customer` vs `User` vs `Account` – sind das dieselben Dinge?)

- **Aktionen** (z.B. `cancel` vs `delete` vs `archive` – was ist der Unterschied?)

- **Zustände** (z.B. `active`, `pending`, `closed` – was bedeutet jedes genau?)

- **Ereignisse** (z.B. `OrderPlaced`, `PaymentReceived`, `ItemShipped`)

  

## Zusammenhang mit Bounded Contexts

  

In DDD kann dasselbe Wort in verschiedenen **Bounded Contexts** unterschiedlich bedeuten:

  

```

Im Kontext "Verkauf": Customer = Person, die kauft

Im Kontext "Support": Customer = Person, die ein Ticket eröffnet hat

Im Kontext "Buchhaltung": Customer = Debitor mit Kontonummer

```

  

Jeder Context hat seine eigene Ubiquitous Language. Wichtig: die Grenzen explizit machen.

  

## Warum es wichtig ist

  

- Weniger Missverständnisse zwischen Tech und Business

- Code ist leichter lesbar für alle Beteiligten

- Bugs durch falsche Interpretation werden seltener

- Neuen Teammitgliedern fällt das Einarbeiten leichter

  

## Weiterführend

  

- Eric Evans – *Domain-Driven Design* (das Originalbuch)

- Vaughn Vernon – *Implementing Domain-Driven Design* (praxisnäher)

- *Domain Modeling Made Functional* (F#, aber sehr gut für funktionale Ansätze)