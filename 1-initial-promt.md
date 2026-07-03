Create a full-stack web application with the following specifications:

## Repository Structure

```
/code       → all application source code (frontend + backend)
/specs      → empty folder (reserved for specification documents)
/harness    → empty folder (reserved for test harness and guidelines)
```

## Code Structure

```
/code
  /backend          → ASP.NET Core Web API (.NET 10.0) (with sln. - file)
  /frontend         → React + Vite SPA
```

## Tech Stack
- **Frontend:** React (Vite) with Untitled UI component library and Tailwind CSS
- **Backend:** ASP.NET Core 10.0 Web API (C#)
- **Database:** SQLite with Entity Framework Core (code-first, Fluent API mappings, Migrations)
- **Architecture:** 2-Tier (React SPA → REST API → SQLite)

---

## Backend Specifications

### Project Setup
- ASP.NET Core Web API, .NET 10.0
- NuGet packages: `Microsoft.EntityFrameworkCore`, `Microsoft.EntityFrameworkCore.Sqlite`,
  `Microsoft.EntityFrameworkCore.Design`
- Enable CORS for `http://localhost:5173` (Vite dev server)
- Swagger/OpenAPI enabled in development

### Domain Model

**Category**
- `Id` (int, PK, auto-increment)
- `Name` (string, required, max 100)

**Person**
- `Id` (int, PK, auto-increment)
- `Name` (string, required, max 200)
- `Age` (int, required, range 0–150)
- `CategoryId` (int, FK → Category)
- Navigation property: `Category`

### Entity Framework Core
- `AppDbContext` with `DbSet<Person>` and `DbSet<Category>`
- **ALL** table mappings via Fluent API in separate `IEntityTypeConfiguration<T>` classes:
  - `PersonConfiguration : IEntityTypeConfiguration<Person>`
  - `CategoryConfiguration : IEntityTypeConfiguration<Category>`
- Applied via `modelBuilder.ApplyConfigurationsFromAssembly()`
- Initial migration + seed data:
  - 3 categories: "Developer", "Designer", "Manager"
  - 5 persons distributed across categories
- Connection string in `appsettings.json`: `"Data Source=app.db"`

### REST API Endpoints

**Categories** (`/api/categories`)
- `GET /api/categories` → list all
- `GET /api/categories/{id}` → single
- `POST /api/categories` → create
- `PUT /api/categories/{id}` → update
- `DELETE /api/categories/{id}` → delete (reject if persons exist in category)

**Persons** (`/api/persons`)
- `GET /api/persons` → list all, include Category name
- `GET /api/persons/{id}` → single, include Category
- `POST /api/persons` → create
- `PUT /api/persons/{id}` → update
- `DELETE /api/persons/{id}` → delete

Use DTOs (request/response) — never expose EF entities directly.
Return proper HTTP status codes (200, 201, 204, 400, 404, 409).

---

## Frontend Specifications

### Setup
- React 18+ with Vite
- Tailwind CSS
- Untitled UI component patterns (use their HTML/CSS class conventions for
  layout, cards, tables, modals, badges, buttons, form inputs)
- Axios for API calls, base URL `http://localhost:5000`

### Pages & Features

**Persons Page (default/home)**
- Table with columns: Name, Age, Category (as colored badge), Actions
- "New Person" button → opens modal form
- Edit icon → opens pre-filled modal form
- Delete icon → confirmation dialog
- Modal form fields: Name (text), Age (number), Category (dropdown, loaded from API)

**Categories Page**
- Table with columns: ID, Name, Actions
- "New Category" button → opens modal form
- Edit / Delete actions (delete blocked with toast error if persons exist)

**Navigation**
- Sidebar or top navbar with links: "Persons" | "Categories"
- Active link highlighted

### UX Details
- Loading spinners during API calls
- Toast notifications for success/error feedback
- Empty states when tables have no data
- Form validation (required fields, age 0–150)

---

## Additional Requirements
- `README.md` in the repository root describing the project structure and tech stack
- `.gitignore` for both .NET and Node
- No authentication required
- All code in English (variable names, comments); UI labels in English