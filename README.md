# 🏛️ architecture-fundamentals

> A personal study repository tracing the evolution of software architecture patterns — from the origins of Hexagonal Architecture to Clean Architecture and beyond.

---

## 📖 Why This Repo?

While preparing for a job interview that required knowledge of **Hexagonal Architecture**, I realized something interesting: the architecture I had been building all along — Clean Architecture — was essentially born *from* Hexagonal. This repo documents that journey of understanding.

---

## 🕰️ The History

### 2005 — Hexagonal Architecture (Ports & Adapters)
**Author:** Alistair Cockburn

Cockburn was frustrated with applications that mixed business logic with UI and database code, making them impossible to test without spinning up the entire system. His insight was simple but powerful:

> *Isolate the core of your application from the outside world. Let everything external connect through well-defined contracts.*

He named it "Hexagonal" simply because of the visual representation he used — a hexagon with the application at the center and adapters on the edges. The number of sides doesn't matter; what matters is the concept of **inside vs. outside**.

The main motivation was **testability**: being able to run the application "headless", without a real database or UI, using mocks at the boundaries.

---

### 2008 — Onion Architecture
**Author:** Jeffrey Palermo

Palermo built on Cockburn's ideas and formalized the internal layers more explicitly. The "onion" metaphor emphasized that layers wrap around a central domain model, and dependencies always flow **inward** — outer layers know about inner layers, but never the reverse.

---

### 2012 / 2017 — Clean Architecture
**Author:** Robert C. Martin (Uncle Bob)

Uncle Bob published his blog post in 2012 and later the book in 2017. He explicitly acknowledged Hexagonal and Onion as inspirations and unified the concepts under a single, more prescriptive framework — naming each layer, each responsibility, and formalizing the **Dependency Rule**.

> All three share the same fundamental principle: **dependencies always point inward, toward the domain.**

---

## 🔑 Core Concepts of Hexagonal Architecture

### The Three Key Concepts

**Port** = an interface that the domain/application *exposes or requires*. It's a contract. The core defines it; it doesn't know who will fulfill it.

**Driving Adapter (Primary / Input)** = whoever *initiates the conversation* with the application. It calls the application's ports. Examples: REST API, CLI, message queue consumer, scheduled job.

**Driven Adapter (Secondary / Output)** = whoever *the application calls* to get things done. It implements the application's ports. Examples: database repository, external HTTP service, email sender, cache.

```
┌─────────────────────────────────────────────────────┐
│                   OUTSIDE WORLD                     │
│                                                     │
│   [REST API]  [CLI]  [Queue]     ← Driving Adapters │
│        │        │       │                           │
│        └────────┴───────┘                           │
│                 │                                   │
│            [ PORT (in) ]                            │
│                 │                                   │
│   ┌─────────────▼─────────────┐                     │
│   │       APPLICATION CORE    │                     │
│   │   (Use Cases + Domain)    │                     │
│   └─────────────┬─────────────┘                     │
│                 │                                   │
│            [ PORT (out) ]                           │
│                 │                                   │
│        ┌────────┴───────┐                           │
│        │                │                           │
│   [Database]      [Email Service]  ← Driven Adapters│
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 🧅 How It Maps to Clean Architecture

If you already know Clean Architecture, you already know Hexagonal. Here's the mapping:

| Clean Architecture | Hexagonal Equivalent |
|---|---|
| `Domain` layer | Application Core (innermost) |
| `Application` layer (interfaces) | **Ports** |
| `Infrastructure` layer (implementations) | **Driven Adapters** (output) |
| `Presentation / API` layer | **Driving Adapters** (input) |

The structure I use:

```
src/
├── Domain/          → Pure domain: entities, value objects, business rules
│                      No NuGet packages. No external dependencies. Ever.
│
├── Application/     → Use cases, interfaces (Ports), DTOs, mappers
│                      References Domain only. No infrastructure concerns.
│
├── Infrastructure/  → Driven Adapters: EF Core, HTTP clients, Redis, etc.
│                      Implements the interfaces defined in Application.
│
└── API/             → Driving Adapters: Controllers, middleware, DI setup
                       References Application only.
```

**The golden rule:** if you open the `Domain` project and see a reference to EF Core, ASP.NET, or any infrastructure NuGet — something is wrong.

---

## 🔁 The Evolution in One Picture

```
Cockburn (2005)          Palermo (2008)          Uncle Bob (2012/2017)
                                                 
  ┌──────────┐           ┌──────────┐            ┌──────────────────┐
  │          │           │  Infra   │            │   Frameworks     │
  │  CORE  ◄─┼─adapters  │ ┌──────┐ │            │ ┌──────────────┐ │
  │          │           │ │ App  │ │            │ │  Interface   │ │
  └──────────┘           │ │┌────┐│ │            │ │  Adapters    │ │
                         │ ││Dom ││ │            │ │ ┌──────────┐ │ │
  "inside vs outside"    │ │└────┘│ │            │ │ │Use Cases │ │ │
                         │ └──────┘ │            │ │ │┌────────┐│ │ │
                         └──────────┘            │ │ ││Entities││ │ │
                                                 │ │ │└────────┘│ │ │
                         "layers that wrap"      │ │ └──────────┘ │ │
                                                 │ └──────────────┘ │
                                                 └──────────────────┘
                                                 
                                                 "named layers + rules"
```

---

## 💡 Key Takeaway

The core insight that connects all three:

> **The business logic should not know — and should not care — about databases, HTTP, frameworks, or any delivery mechanism. It should be testable in complete isolation.**

Hexagonal gave us the *vocabulary*. Onion gave us the *layers*. Clean Architecture gave us the *rules*.

---

## 🧪 Testability: The Original Motivation

Because adapters are replaceable, testing becomes trivial:

```csharp
// No database. No HTTP. Just pure business logic.
[Fact]
public async Task ShouldCreateOrderWithValidData()
{
    var repo = new Mock<IOrderRepository>(); // Mock = fake Driven Adapter
    var useCase = new CreateOrderUseCase(repo.Object);

    var id = await useCase.ExecuteAsync("customer-1", 150m);

    Assert.NotEqual(Guid.Empty, id);
    repo.Verify(r => r.SaveAsync(It.IsAny<Order>()), Times.Once);
}
```

This was Cockburn's original goal in 2005 — and it still holds today.

---

## 📚 References

- [Alistair Cockburn — Hexagonal Architecture (2005)](https://alistair.cockburn.us/hexagonal-architecture/)
- [Jeffrey Palermo — Onion Architecture (2008)](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/)
- [Robert C. Martin — Clean Architecture (blog, 2012)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- Clean Architecture — Robert C. Martin (book, 2017)

---

*Study notes by a developer who discovered they already knew Hexagonal Architecture — they just didn't know what to call it.*
