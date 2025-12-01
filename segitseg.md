# ASP.NET Core MVC + EF Core + SQLite ZH – Teljes Maximális Pontszámot Biztosító Útmutató

Ez a dokumentum tartalmazza **a teljes, univerzális, bármilyen EF Core + MVC + SQLite zárthelyire alkalmazható megoldási tervet**, amelynek követésével **a maximális pontszám megszerezhető**. A leírás általános és minden szokásos ZH-feladatra alkalmazható (2 modell, reláció, validációk, CRUD, adatbázis, kép/URL megjelenítése, dropdown, stb.).

---

# 0. Projekt létrehozása

**Visual Studio → Create a new project → ASP.NET Core Web App (Model-View-Controller)**

* Project név: tetszőleges vagy amit kérnek
* Framework: .NET 6/7/8
* Create

## Szükséges NuGet csomagok (3 db):

Telepítsd:

* `Microsoft.EntityFrameworkCore`
* `Microsoft.EntityFrameworkCore.Sqlite`
* `Microsoft.EntityFrameworkCore.Tools`

(A `Microsoft.EntityFrameworkCore.Design` automatikusan települ, ha szükséges.)

---

# 1. appsettings.json

A fájl tartalmát **nem töröljük ki**, csak kiegészítjük:

```json
"ConnectionStrings": {
  "DefaultConnection": "Data Source=database.db"
}
```

A többi rész marad.

---

# 2. ApplicationDbContext (ZH 1. feladat – 5 pont)

📁 **Data mappa → ApplicationDbContext.cs**

```csharp
using Microsoft.EntityFrameworkCore;
using YourProject.Models;

namespace YourProject.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options) { }

        // DbSet = táblák az adatbázisban
        public DbSet<Category> Categories { get; set; }
        public DbSet<Pet> Pets { get; set; }
    }
}
```

---

# 3. Program.cs – DbContext regisztrálása

A `builder.Services.AddControllersWithViews()` fölé kerül:

```csharp
using Microsoft.EntityFrameworkCore;
using YourProject.Data;

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

---

# 4. Modellek létrehozása (ZH 2–3. feladat – 8–10 pont)

📁 **Models mappa → Add → Class**

## Category példa

```csharp
using System.ComponentModel.DataAnnotations;

public class Category
{
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; } = "";
}
```

## Pet példa (idegen kulccsal)

```csharp
using System.ComponentModel.DataAnnotations;

public class Pet
{
    public int Id { get; set; }

    [Required, StringLength(100)]
    public string Name { get; set; } = "";

    [Range(0,200)]
    public int Age { get; set; }

    [Range(0,1000)]
    public float Weight { get; set; }

    [Required]
    [RegularExpression(@"^.*\.(png|jpg)$")]
    public string PhotoPath { get; set; } = "";

    // IDEGEN KULCS
    public int CategoryId { get; set; }
    public Category? Category { get; set; }
}
```

## Gyakran használt annotációk

```csharp
[Required]
[StringLength(n)]
[Range(min,max)]
[EmailAddress]
[Url]
[Display(Name="...")]
[RegularExpression("...")]
```

---

# 5. Migráció és adatbázis létrehozása

**Tools → NuGet Package Manager → Package Manager Console**

```
Add-Migration Initial
Update-Database
```

A `database.db` létrejön.

---

# 6. CRUD Generálás (Scaffolding) – ZH 4. feladat

📁 Controllers → **Add → New Scaffolded Item** →
**MVC Controller with views using EF**

Generáld le:

* Category CRUD-ot
* Pet CRUD-ot

Ez létrehozza: Controller + Views (Index, Create, Edit, Delete, Details)

---

# 7. Navigáció hozzáadása (_Layout.cshtml)

📁 `Views/Shared/_Layout.cshtml`

Keresd:

```html
<div class="navbar-nav">
```

Alá írd:

```html
<a class="nav-link text-dark" asp-controller="Categories" asp-action="Index">Categories</a>
<a class="nav-link text-dark" asp-controller="Pets" asp-action="Index">Pets</a>
```

---

# 8. Kötelező finomítások (ZH 5. feladat – 5–7 pont)

## 8.1 Lebegőpontos mező Create/Edit nézetben

`Views/Pets/Create.cshtml` és `Edit.cshtml`:

```html
<input asp-for="Weight" class="form-control" type="number" step="0.01" />
```

---

## 8.2 Kapcsolt elem neve jelenjen meg (ne az ID)

`Views/Pets/Index.cshtml`:

```html
@item.Category?.Name
```

---

## 8.3 Kép megjelenítése a Details oldalon

`Views/Pets/Details.cshtml`:

```html
<img src="@Model.PhotoPath" style="max-width:300px; max-height:200px;" />
```

---

## 8.4 Dropdown idegen kulcshoz Create/Edit oldalon

A Scaffold ezt általában megcsinálja, de így kell kinéznie:

```html
<select asp-for="CategoryId" class="form-control" asp-items="ViewBag.CategoryId"></select>
```

---

# 9. Beadás (ZIP készítése)

**ZIP-be KELL:**

* Controllers
* Models
* Views
* Data
* Migrations
* `database.db`
* `Program.cs`
* `appsettings.json`

**NE TEDD BE:**

* `bin/`
* `obj/`
* `.vs/`

---

# 10. Összefoglalás – Mikor lesz maximum pontod?

✔ Modellek teljesek + validációk
✔ DbContext + Program.cs beállítva
✔ Migrációk rendben
✔ CRUD működik mindkét modellre
✔ Kép megjelenítés
✔ Dropdown idegen kulccsal
✔ Navigáció megvan
✔ Minden nézet helyesen működik

---

# ALTERNATÍV MEGOLDÁS: RÉGI STÍLUSÚ (ONCONFIGURING-ES) EF CONTEXT ZH-HOZ

Ez a szekció egy teljesen működő, **óra szerint tanított**, régebbi EF Core mintát követő megoldás, amely **akkor is működik**, ha a modern Program.cs-es + appsettings-es megoldás valamilyen okból nem futna el a kabinetes gépen.

Használhatod B-tervként, ha a modern megoldás nem működik.

---

# 🔵 0. Projekt létrehozása (ugyanaz mint a modern megoldásban)

ASP.NET Core Web App (Model-View-Controller)

NuGet-ek (óra szerint, ahogy tanultátok):

* Microsoft.EntityFrameworkCore (9.0.10)
* Microsoft.EntityFrameworkCore.Sqlite (9.0.10)
* Microsoft.EntityFrameworkCore.Tools (9.0.10)
* Microsoft.EntityFrameworkCore.Design (9.0.10) – opcionális

---

# 🔵 1. appsettings.json NEM kell ehhez a módszerhez

Ebben a módszerben **nem használunk** appsettings.json connection stringet.
A connection string a Context `OnConfiguring` metódusába kerül.

(appsettings.json maradhat érintetlen, nem fog zavarni.)

---

# 🔵 2. Régi stílusú EFContext létrehozása (ONCONFIGURING-GEL)

📁 Add → New Folder: **Context**
📁 Add → Class → **EFContext.cs**

```csharp
using Microsoft.EntityFrameworkCore;
using YourProject.Models;

namespace YourProject.Context
{
    public class EFContext : DbContext
    {
        // DbSet-ek (táblák)
        public virtual DbSet<Category> Categories { get; set; }
        public virtual DbSet<Pet> Pets { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            // Itt adjuk meg az adatbázis helyét
            // Ez a módszer közvetlenül beégeti a connection stringet
            optionsBuilder.UseSqlite("Data Source=allatok.db");
        }
    }
}
```

👉 A connection string **a kódban van**, nincs szükség Program.cs beállításra.
👉 A ZH-n ez tökéletesen működik.
👉 Ugyanúgy generálható migráció és Scaffold.

---

# 🔵 3. Program.cs minimális módosítása

Az órai régi stílusú megoldás alapján csak ezt kell belerakni:

```csharp
builder.Services.AddDbContext<EFContext>();
```

📌 **Fontos:**
Ebben a B-tervben **nem teszünk ide UseSqlite-et**, mert azt az EFContext-ben már megadtuk.

---

# 🔵 4. Modellek létrehozása (ugyanaz mint modern megoldásban)

Nincs különbség.

---

# 🔵 5. Migrációk (ugyanaz a parancs, mindkét módszernél működik)

Package Manager Console:

```
Add-Migration Initial
Update-Database
```

A `allatok.db` létrejön.

---

# 🔵 6. Scaffold (ugyanaz mint a modern megoldásnál)

Controllers → Add → New Scaffolded Item →
**MVC Controller with views using Entity Framework**

* Model: Category / Pet
* DbContext: **EFContext** (NEM ApplicationDbContext)

A Scaffold hibátlanul működik ezzel a régi stílussal is.

---

# 🔵 7. Navigáció (_Layout.cshtml)

Ugyanaz mint a modern megoldásnál:

```html
<li class="nav-item">
    <a class="nav-link text-dark" asp-controller="Pets" asp-action="Index">Pets</a>
</li>
```

---

# 🔵 8. Nézetek módosítása – teljesen ugyanaz mint a modern megoldásban

* lebegőpontos mező → step=0.01
* kapcsolt elem neve → @item.Category?.Name
* kép megjelenítése → <img src="...">
* dropdown → asp-items="ViewBag.CategoryId"

A B-terv **csak a DbContext és a connection string kezelésében más**, minden más lépés identikus.

---

# 🔵 9. Mikor használd ezt az alternatív megoldást?

Használd ezt, ha:

* a kabinetes gépen valamiért nem működik a modern Program.cs-es regisztráció,
* vagy a scaffold hibát dob,
* vagy a tanár kifejezetten *óra szerinti* megoldást vár,
* vagy egyszerűen gyorsabbnak érzed.

---

# 🔵 10. Összefoglalás – különbségek röviden

| Modern megoldás            | Régi megoldás (OnConfiguring)   |
| -------------------------- | ------------------------------- |
| Program.cs-ben UseSqlite   | Connection string a Context-ben |
| appsettings.json használva | appsettings.json nem kell       |
| Ipari standard             | Órai / egyszerűbb minta         |
| Tiszta, bővíthető          | Gyors, egyszerű                 |
| Ajánlott                   | Elfogadott                      |

Mindkét módszer TELJES MÉRTÉKBEN elfogadható a ZH-n.

A modern jobb és tisztább, de a régi biztosan működik, ha valami gebasz lenne.

---

# 🔵 11. Ajánlás

Használd a modern megoldást.

Ha valamiért baj van → válts át erre az alternatív OnConfiguring-es EFContext változatra.
