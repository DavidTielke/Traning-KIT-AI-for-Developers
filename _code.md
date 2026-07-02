# C# Coding Guideline — ReSharper Default Conventions

This guideline describes the **default code style of JetBrains ReSharper** for C#.
All C# source code in this repository (currently `/code/backend`) MUST be written
so that it produces **zero ReSharper warnings or suggestions with out-of-the-box
settings**. AI agents must apply these rules from the first line of code they write —
not as an afterthought or cleanup step.

---

## 1. Naming Conventions

| Element                                   | Style              | Example                       |
|-------------------------------------------|--------------------|-------------------------------|
| Namespaces                                 | `UpperCamelCase`   | `Backend.Data.Configurations` |
| Types (class, struct, enum, delegate)      | `UpperCamelCase`   | `PersonConfiguration`         |
| Interfaces                                 | `I` + `UpperCamelCase` | `IEntityTypeConfiguration` |
| Type parameters                            | `T` + `UpperCamelCase` | `TEntity`, `TResult`       |
| Methods                                    | `UpperCamelCase`   | `GetPersons()`                |
| Properties                                 | `UpperCamelCase`   | `CategoryId`                  |
| Events                                     | `UpperCamelCase`   | `DataChanged`                 |
| Public fields                              | `UpperCamelCase`   | `MaxRetries`                  |
| Private/protected/internal instance fields | `lowerCamelCase` (leading `_` also accepted) | `context` / `_context` |
| Constant fields                            | `UpperCamelCase`   | `DefaultPageSize`             |
| Static readonly fields                     | `UpperCamelCase`   | `EmptyResult`                 |
| Enum members                               | `UpperCamelCase`   | `OrderState.Shipped`          |
| Local variables                            | `lowerCamelCase`   | `personCount`                 |
| Local constants                            | `lowerCamelCase`   | `maxAge`                      |
| Parameters                                 | `lowerCamelCase`   | `categoryId`                  |
| Async methods (returning `Task`/`Task<T>`) | suffix `Async`     | `LoadDataAsync()`             |

Additional naming rules:
- No Hungarian notation, no `m_` prefixes, no ALL_CAPS constants.
- Abbreviations of two letters stay uppercase (`Id` is the exception: `PersonId`, not `PersonID`).
- Event handlers: `On` + event name or `<Subject>_<Event>` only in generated code.

## 2. File & Type Organization

- **One top-level type per file**; file name matches the type name (`Person.cs` → `class Person`).
- Namespace must **match the folder structure** relative to the project root
  (`Data/Configurations/PersonConfiguration.cs` → `Backend.Data.Configurations`).
- Prefer **file-scoped namespaces** (`namespace Backend.Models;`).
- `using` directives:
  - placed **at the top of the file, outside the namespace**,
  - `System.*` namespaces **sorted first**, then all others alphabetically,
  - **no unused usings** (ReSharper flags them immediately).
- Member order inside a type: fields → constructors → properties → methods (public before private is not enforced, but keep a consistent, logical order).

## 3. Formatting

- **Indentation: 4 spaces**, never tabs.
- **Braces: BSD/Allman style** — opening brace on its **own line** for *all* constructs
  (types, methods, properties, `if`, `for`, `while`, `try`, lambdas with block bodies):

  ```csharp
  public class Person
  {
      public int GetAge()
      {
          if (birthDate == default)
          {
              return 0;
          }

          return CalculateAge(birthDate);
      }
  }
  ```

- Maximum line length: **120 characters** (ReSharper default wrap margin); wrap longer lines.
- At most **one consecutive blank line** inside members; at most two between members.
- Spaces:
  - after control-flow keywords: `if (condition)`, `for (...)`, `while (...)`
  - **no** space between method name and parenthesis: `DoWork()`
  - around binary operators: `a + b`, `x == null`
  - after commas, none before: `Method(a, b, c)`
- One statement per line; one declaration per line.

## 4. Language & Syntax Style (ReSharper suggestions)

These are the styles ReSharper actively suggests with default settings — code must
already conform so no squiggles appear:

- **`var` everywhere**: use `var` for local variable declarations, including
  built-in types, whenever the type is evident from the right-hand side:
  ```csharp
  var count = 5;
  var persons = new List<Person>();
  var category = await context.Categories.FindAsync(id);
  ```
- **Built-in type keywords**, never CLR type names: `int`, `string`, `bool` — not `Int32`, `String`, `Boolean`.
- **Explicit access modifiers** on all types and members (`private` is written out, not implied).
- **Modifier order** (C# canonical order enforced by ReSharper):
  `public/protected/internal/private` → `new` → `abstract/virtual/override/sealed` → `static` → `readonly` → `extern` → `unsafe` → `volatile` → `async`.
- **No redundant `this.` qualifier** — only use it when required for disambiguation.
- **No redundant code** (ReSharper greys these out):
  - no unused parameters/variables, no redundant casts, no redundant parentheses,
  - no redundant base constructor calls, no empty argument lists on object initializers,
  - no redundant `else` after `return`/`throw`/`continue`/`break`.
- **Object & collection initializers** instead of sequential assignments:
  ```csharp
  var person = new Person { Name = "Alice", Age = 29 };
  ```
- **Expression-bodied members** for simple single-expression properties and accessors;
  regular block bodies for methods with logic:
  ```csharp
  public string DisplayName => $"{Name} ({Age})";
  ```
- **String interpolation** instead of `string.Format` or concatenation:
  `$"Category {id} not found"`.
- **`nameof(...)`** instead of string literals for symbol names.
- **Null-propagation and null-coalescing** where applicable: `person?.Category?.Name`, `value ?? fallback`.
- **Pattern matching** over `as` + null check: `if (obj is Person person)`.
- Prefer **`is null` / `is not null`** or `== null` consistently (do not mix styles in one file).
- **Invert `if` to reduce nesting** where it improves readability (ReSharper suggests early returns).
- Use **LINQ methods** where they simplify loops (ReSharper suggests e.g. `Where`/`Select`/`Any` conversions).

## 5. Async & Error Handling

- Async all the way: never block with `.Result` or `.Wait()`.
- Methods returning `Task`/`Task<T>` end with `Async` (except controller actions, entry points, and test methods where the framework dictates names).
- Do not catch exceptions without handling them; never swallow with an empty `catch`.

## 6. Comments & Documentation

- Comments in **English**, explaining *why*, not *what*.
- XML doc comments (`///`) for public APIs where the intent is not obvious from the signature.
- No commented-out code in committed files.
- `// TODO:` only with a concrete description.

## 7. Compliance Example

A fully conformant snippet in this codebase looks like:

```csharp
using Microsoft.EntityFrameworkCore;

namespace Backend.Services;

public class PersonService
{
    private readonly AppDbContext context;

    public PersonService(AppDbContext context)
    {
        this.context = context;
    }

    public async Task<IReadOnlyList<Person>> GetAdultsAsync(int minimumAge)
    {
        var adults = await context.Persons
            .AsNoTracking()
            .Where(p => p.Age >= minimumAge)
            .OrderBy(p => p.Name)
            .ToListAsync();

        return adults;
    }
}
```

---

**Rule of thumb:** if opening the file in Rider/ReSharper with default settings would
show any squiggle, greyed-out code, or "Code cleanup" change — the code is not compliant.
