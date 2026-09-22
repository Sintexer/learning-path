**"Favor composition over inheritance"** means: when you want to reuse behavior or extend functionality, prefer **building an object out of smaller collaborating objects** (composition — a "has-a" relationship) rather than **extending a base class** (inheritance — an "is-a" relationship), especially when the relationship is really about _behavior_ rather than _identity_.

This is not "inheritance is always bad" — it's "inheritance is a specific, powerful, but rigid tool that's frequently reached for when a more flexible tool would serve better."

## Why not Inheritance

### 1. Fragile Base Class Problem

If you change behavior in a base class, **every subclass inherits that change** — sometimes in ways the subclass author never anticipated. A seemingly safe change deep in a hierarchy can silently break behavior several levels down, and it's not obvious at the call site that this coupling even exists. The dependency is invisible unless you go read the parent class.

### 2. Rigid hierarchy, decided at compile time

Inheritance relationships are fixed once compiled — an `EmailSender` that extends `NotificationSender` is locked into that relationship. Composition lets you **swap behavior at runtime** — inject a different collaborator, change behavior dynamically, without recompiling a new class for every combination.

### 3. Single inheritance limitation (Java/Kotlin specifically)

You can only extend **one** class. If a class needs to combine multiple independent behaviors (say, "retryable" + "loggable" + "rate-limited"), inheritance forces awkward workarounds. Composition has no such limit — you can inject as many collaborators as you want.

### 4. Testability

A class built from injected, composed dependencies can have those dependencies trivially replaced with mocks/fakes in a test. A class buried three levels deep in an inheritance chain often drags along behavior from every ancestor, making it much harder to isolate what you're actually testing.