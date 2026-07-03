# Backend Design Contract — People Manager

Binding design decisions derived from the implemented backend (`/code/backend`). Every new feature MUST follow these patterns exactly. Code style: see `harness/code.md` (ReSharper defaults). `{X}` = entity singular, `{Xs}` = plural, `{Y}` = related principal entity.

## 1. Architecture

- 2-tier: Controller → `AppDbContext` (EF Core) → SQLite. **No** service/repository/mediator/UoW layers, no AutoMapper, no interfaces for application classes, no custom middleware/filters/exception handlers.
- Namespaces = folders: `Backend.Controllers`, `Backend.Data`, `Backend.Data.Configurations`, `Backend.Dtos`, `Backend.Models`.
- A new entity consists of **exactly 4 files** plus wiring:
  1. `Models/{X}.cs` · 2. `Dtos/{X}Dtos.cs` · 3. `Data/Configurations/{X}Configuration.cs` · 4. `Controllers/{Xs}Controller.cs`
  + one `DbSet` line in `AppDbContext` + an EF migration (`dotnet ef migrations add <Name>`).
- `Program.cs` (top-level statements) already handles: DI, SQLite connection (`DefaultConnection`), CORS (policy `FrontendCorsPolicy`, only `http://localhost:5173`), Swagger (dev only), auto-migration at startup (`db.Database.Migrate()`). New controllers need **no** registration; only touch `Program.cs` for genuinely new infrastructure.

## 2. Entities (`Models/{X}.cs`)

Plain POCOs, **zero attributes** — mapping lives exclusively in configurations:

```csharp
public class Person
{
    public int Id { get; set; }                       // int PK, always first
    public string Name { get; set; } = string.Empty;  // strings default to string.Empty
    public int CategoryId { get; set; }               // FK: int {Y}Id ...
    public Category Category { get; set; } = null!;   // ...+ required nav = null!
}
```

- Principal side gets the inverse collection: `public ICollection<{X}> {Xs} { get; set; } = new List<{X}>();`
- No behavior, no constructors, no computed properties.

## 3. DTOs (`Dtos/{X}Dtos.cs`)

One file per entity containing both types; entities never leave the controller:

- **Response**: `public record {X}Response(int Id, string Name, ..., int {Y}Id, string {Y}Name);` — positional record; carries the FK id **and** the related display name (e.g. `CategoryName`) so clients need no second call.
- **Request**: mutable class with DataAnnotations, no `Id` property:

```csharp
public class PersonRequest
{
    [Required]
    [MaxLength(200)]
    public string Name { get; set; } = string.Empty;

    [Required]
    [Range(0, 150)]
    public int Age { get; set; }

    [Required]
    public int CategoryId { get; set; }
}
```

- Validation attributes (`[Required]`, `[MaxLength]`, `[Range]`) are the single source of request validation; `[ApiController]` turns violations into automatic 400 responses. Limits MUST match the EF configuration.
- One request type is reused for both create and update (no separate Create/Update DTOs).

## 4. EF configuration (`Data/`)

- `AppDbContext`: constructor takes `DbContextOptions<AppDbContext>`; sets are expression-bodied — `public DbSet<{X}> {Xs} => Set<{X}>();`; `OnModelCreating` calls `base` then `ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly)` — never add per-entity code there.
- Per entity: `public class {X}Configuration : IEntityTypeConfiguration<{X}>` in `Data/Configurations/`, statements in this order inside `Configure`:
  1. `builder.ToTable("{Xs}");` (PascalCase plural)
  2. `builder.HasKey(x => x.Id);`
  3. `builder.Property(x => x.Id).ValueGeneratedOnAdd();`
  4. Per property: `.IsRequired()` / `.HasMaxLength(n)` — identical limits as the DTO annotations.
  5. Relationships configured from the **dependent** side: `builder.HasOne(x => x.{Y}).WithMany(y => y.{Xs}).HasForeignKey(x => x.{Y}Id).OnDelete(DeleteBehavior.Restrict);` — always `Restrict`, never cascade.
  6. `// Seed data` comment + `builder.HasData(...)` with explicit ids; every entity ships a handful of seed rows.

## 5. Controllers (`Controllers/{Xs}Controller.cs`)

Class shape: `[ApiController]`, `[Route("api/{xs}")]` (lowercase plural, literal string — no `[controller]` token), `: ControllerBase`; single constructor injecting `AppDbContext` into `private readonly AppDbContext _context;`. No other dependencies, no logging.

Exactly these 5 actions per resource — all `async`/`await`, **no** `Async` name suffix, id-routes constrained `"{id:int}"`:

| Action | Route | Signature | Flow |
|---|---|---|---|
| `Get{Xs}()` | `GET` | `Task<ActionResult<IEnumerable<{X}Response>>>` | `AsNoTracking()` → `Include` needed navs → `OrderBy(x => x.Id)` → `Select` projection to DTO → `ToListAsync()` → `Ok(list)` |
| `Get{X}(int id)` | `GET {id}` | `Task<ActionResult<{X}Response>>` | `AsNoTracking()` + `Include` → `FirstOrDefaultAsync` → null ⇒ `NotFound()` → `Ok(new {X}Response(...))` |
| `Create{X}(request)` | `POST` | `Task<ActionResult<{X}Response>>` | FK check ⇒ 400 → `new {X} { ... }` from request → `Add` → `SaveChangesAsync` → `CreatedAtAction(nameof(Get{X}), new { id = x.Id }, response)` |
| `Update{X}(int id, request)` | `PUT {id}` | `Task<ActionResult<{X}Response>>` | `FindAsync(id)` ⇒ 404 → FK check ⇒ 400 → mutate props → `SaveChangesAsync` → `Ok(response)` |
| `Delete{X}(int id)` | `DELETE {id}` | `Task<IActionResult>` | `FindAsync(id)` ⇒ 404 → dependent check (`AnyAsync`) ⇒ 409 → `Remove` → `SaveChangesAsync` → `NoContent()` |

**Sanctioned exception — manual ordering:** an entity with a user-controlled order (currently only `Person`, via `SortOrder`) additionally exposes `Update{X}Position(int id, {X}PositionRequest request)` on `PUT "{id:int}/position"`. The request carries `TargetIndex` (zero-based position in the ordered list, validated via `[Range(0, int.MaxValue)]` and clamped server-side to the list bounds); the action loads all rows ordered by `SortOrder`, moves the entity to the target index, renumbers `SortOrder` sequentially (1-based) and returns **204**. The auxiliary `{X}PositionRequest` lives in the same `Dtos/{X}Dtos.cs` file. New persons get `SortOrder = max + 1` on create.

Method-level rules:
- **Reads** project to DTOs inside the query (`Select(x => new {X}Response(...))`); **writes** load via `FindAsync` and build the response DTO manually. Never return tracked entities or anonymous success bodies.
- Null checks use pattern matching with braces: `if (x is null) { return NotFound(); }` (404 has no body).
- **FK validation** precedes every write, and the loaded principal is reused for the response name:
  ```csharp
  var category = await _context.Categories.FindAsync(request.CategoryId);
  if (category is null)
  {
      return BadRequest(new { message = $"Category with id {request.CategoryId} does not exist." });
  }
  ```
- **Referential guard** on principal delete: `await _context.{Xs}.AnyAsync(x => x.{Y}Id == id)` ⇒ `Conflict(new { message = "Cannot delete {y} because {xs} are assigned to it." })`.
- Error bodies are always anonymous objects `new { message = "..." }` — full English sentences ending with `.`. No ProblemDetails, no error codes.
- Request DTOs are plain parameters (implicit `[FromBody]`); no `ModelState` checks (handled by `[ApiController]`); no try/catch (no expected exceptions).

## 6. API conventions

- Routes: `api/{plural-lowercase}`; JSON camelCase (framework default); responses are bare DTOs/arrays — no envelope, no HATEOAS.
- Status codes: **200** GET/PUT · **201** POST via `CreatedAtAction` (Location header) · **204** DELETE · **400** validation (automatic + manual FK checks) · **404** unknown id · **409** referential conflict.
- List endpoints return everything ordered by `Id` (entities with manual ordering: by `SortOrder`, currently only `Person`) — **no** paging, filtering, sorting, search params, auth, or versioning. Do not introduce any of these unprompted.
- Schema changes always go through a migration; never edit existing migrations or the snapshot by hand.
