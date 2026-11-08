# Razor-Layout-Partial — 2-апта тапсырмасы

ASP.NET Core MVC (.NET 8): ProductCatalog (тапсырма) және Lab02_StudentApp (зертханалық жұмыс).

Іске қосу: `cd ProductCatalog && dotnet run` → `http://localhost:5070/Product`

## `.gitignore`

````
bin/
obj/
.vs/
*.user
````

## `ProductCatalog/ProductCatalog.csproj`

````xml

<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <RootNamespace>ProductCatalog</RootNamespace>
  </PropertyGroup>

</Project>
````

## `ProductCatalog/Program.cs`

````csharp

using ProductCatalog.Data;

var builder = WebApplication.CreateBuilder(args);

// MVC сервистерін (Controllers + Views) тіркеу
builder.Services.AddControllersWithViews();

// Тауарлар репозиторийін DI контейнеріне тіркеу (жадтағы деректер)
builder.Services.AddSingleton<IProductRepository, InMemoryProductRepository>();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Product}/{action=Index}/{id?}");

app.Run();
````

## `ProductCatalog/appsettings.json`

````json

{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
````

## `ProductCatalog/Properties/launchSettings.json`

````json

{
  "profiles": {
    "ProductCatalog": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7070;http://localhost:5070",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
````

## `ProductCatalog/Controllers/HomeController.cs`

````csharp

using System.Diagnostics;
using Microsoft.AspNetCore.Mvc;
using ProductCatalog.Models;

namespace ProductCatalog.Controllers
{
    public class HomeController : Controller
    {
        public IActionResult About()
        {
            ViewData["Title"] = "Жоба туралы";
            return View();
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
            return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
        }
    }
}
````

## `ProductCatalog/Controllers/ProductController.cs`

````csharp

using Microsoft.AspNetCore.Mvc;
using ProductCatalog.Data;
using ProductCatalog.Models;

namespace ProductCatalog.Controllers
{
    public class ProductController : Controller
    {
        private readonly IProductRepository _repository;

        public ProductController(IProductRepository repository)
        {
            _repository = repository;
        }

        // GET: /Product  немесе  /Product?category=Кітаптар
        public IActionResult Index(string? category)
        {
            IEnumerable<Product> products = _repository.GetAll();

            if (!string.IsNullOrWhiteSpace(category))
            {
                products = products.Where(p => p.Category == category);
            }

            var list = products.ToList();

            // ViewData — Dictionary<string, object> арқылы деректер беру
            ViewData["Title"] = "Тауарлар каталогы";

            // ViewBag — ViewData үстіндегі dynamic қабықша
            ViewBag.Categories = _repository.GetCategories().ToList();
            ViewBag.SelectedCategory = category;
            ViewBag.TotalCount = list.Count;
            ViewBag.AvailableCount = list.Count(p => p.IsAvailable);

            // Strongly typed View — модельді тікелей беру
            return View(list);
        }

        // GET: /Product/Details/5
        public IActionResult Details(int id)
        {
            var product = _repository.GetById(id);
            if (product == null)
            {
                return NotFound();
            }

            ViewData["Title"] = product.Name;
            return View(product);
        }
    }
}
````

## `ProductCatalog/Data/IProductRepository.cs`

````csharp

using ProductCatalog.Models;

namespace ProductCatalog.Data
{
    public interface IProductRepository
    {
        IEnumerable<Product> GetAll();
        Product? GetById(int id);
        IEnumerable<string> GetCategories();
    }
}
````

## `ProductCatalog/Data/InMemoryProductRepository.cs`

````csharp

using ProductCatalog.Models;

namespace ProductCatalog.Data
{
    /// <summary>
    /// Дерекқорсыз, жадта сақталатын тауарлар тізімі (2-апта тапсырмасы үшін).
    /// </summary>
    public class InMemoryProductRepository : IProductRepository
    {
        private readonly List<Product> _products = new()
        {
            new Product { Id = 1, Name = "Ноутбук Lenovo IdeaPad 5", Price = 389990m, Category = "Электроника", IsAvailable = true },
            new Product { Id = 2, Name = "Смартфон Samsung Galaxy A55", Price = 199990m, Category = "Электроника", IsAvailable = true },
            new Product { Id = 3, Name = "Сымсыз құлаққап", Price = 24990m, Category = "Электроника", IsAvailable = false },
            new Product { Id = 4, Name = "C# 12 and .NET 8 (кітап)", Price = 15500m, Category = "Кітаптар", IsAvailable = true },
            new Product { Id = 5, Name = "Clean Code (кітап)", Price = 12900m, Category = "Кітаптар", IsAvailable = false },
            new Product { Id = 6, Name = "Офис орындығы", Price = 64990m, Category = "Жиһаз", IsAvailable = true },
            new Product { Id = 7, Name = "Жазу үстелі", Price = 79990m, Category = "Жиһаз", IsAvailable = true },
            new Product { Id = 8, Name = "Спорттық рюкзак", Price = 18990m, Category = "Аксессуарлар", IsAvailable = true }
        };

        public IEnumerable<Product> GetAll() => _products;

        public Product? GetById(int id) => _products.FirstOrDefault(p => p.Id == id);

        public IEnumerable<string> GetCategories() =>
            _products.Select(p => p.Category).Distinct().OrderBy(c => c);
    }
}
````

## `ProductCatalog/Models/ErrorViewModel.cs`

````csharp

namespace ProductCatalog.Models
{
    public class ErrorViewModel
    {
        public string? RequestId { get; set; }
        public bool ShowRequestId => !string.IsNullOrEmpty(RequestId);
    }
}
````

## `ProductCatalog/Models/Product.cs`

````csharp

using System.ComponentModel.DataAnnotations;

namespace ProductCatalog.Models
{
    /// <summary>
    /// Каталогтағы тауар моделі.
    /// </summary>
    public class Product
    {
        public int Id { get; set; }

        [Display(Name = "Атауы")]
        public string Name { get; set; } = string.Empty;

        [Display(Name = "Бағасы")]
        [DataType(DataType.Currency)]
        public decimal Price { get; set; }

        [Display(Name = "Санаты")]
        public string Category { get; set; } = string.Empty;

        [Display(Name = "Қоймада бар ма")]
        public bool IsAvailable { get; set; }
    }
}
````

## `ProductCatalog/Views/Home/About.cshtml`

````razor

@{
    ViewData["Title"] = "Жоба туралы";
}
<h1 class="h2">@ViewData["Title"]</h1>
<p class="lead">ProductCatalog — 2-апта тапсырмасы: Razor синтаксисі, деректерді беру, Layout және Partial View.</p>

<div class="table-responsive">
    <table class="table table-bordered align-middle">
        <thead class="table-light">
            <tr><th>Тақырып</th><th>Жобада қолданылуы</th></tr>
        </thead>
        <tbody>
            <tr><td>Strongly Typed View</td><td><code>@@model List&lt;Product&gt;</code> — Product/Index, Product/Details</td></tr>
            <tr><td>ViewData</td><td><code>ViewData["Title"]</code> — бет тақырыбы</td></tr>
            <tr><td>ViewBag</td><td>Санаттар тізімі, тауарлар саны, таңдалған санат</td></tr>
            <tr><td>Layout</td><td><code>Views/Shared/_Layout.cshtml</code> + <code>@@RenderBody()</code></td></tr>
            <tr><td>Partial View</td><td><code>Views/Shared/_ProductCard.cshtml</code> (<code>&lt;partial&gt;</code> tag helper)</td></tr>
        </tbody>
    </table>
</div>
````

## `ProductCatalog/Views/Product/Details.cshtml`

````razor

@model ProductCatalog.Models.Product

<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item"><a asp-action="Index">Каталог</a></li>
        <li class="breadcrumb-item">
            <a asp-action="Index" asp-route-category="@Model.Category">@Model.Category</a>
        </li>
        <li class="breadcrumb-item active" aria-current="page">@Model.Name</li>
    </ol>
</nav>

<div class="card shadow-sm">
    <div class="card-header bg-primary text-white">
        <h2 class="h4 mb-0">@Model.Name</h2>
    </div>
    <div class="card-body">
        <dl class="row mb-0">
            <dt class="col-sm-3">@Html.DisplayNameFor(m => m.Id)</dt>
            <dd class="col-sm-9">@Model.Id</dd>

            <dt class="col-sm-3">@Html.DisplayNameFor(m => m.Category)</dt>
            <dd class="col-sm-9">@Model.Category</dd>

            <dt class="col-sm-3">@Html.DisplayNameFor(m => m.Price)</dt>
            <dd class="col-sm-9 fw-bold text-primary">@Model.Price.ToString("N0") ₸</dd>

            <dt class="col-sm-3">@Html.DisplayNameFor(m => m.IsAvailable)</dt>
            <dd class="col-sm-9">
                @if (Model.IsAvailable)
                {
                    <span class="badge bg-success">Иә</span>
                }
                else
                {
                    <span class="badge bg-danger">Жоқ</span>
                }
            </dd>
        </dl>
    </div>
    <div class="card-footer">
        <a asp-action="Index" class="btn btn-secondary"><i class="bi bi-arrow-left"></i> Каталогқа оралу</a>
    </div>
</div>
````

## `ProductCatalog/Views/Product/Index.cshtml`

````razor

@model List<ProductCatalog.Models.Product>
@{
    var categories = (List<string>)ViewBag.Categories;
    string? selected = ViewBag.SelectedCategory;
}

<div class="d-flex flex-wrap justify-content-between align-items-center mb-4">
    <h1 class="h2 mb-2">@ViewData["Title"]</h1>
    <div class="text-muted">
        Барлығы: <strong>@ViewBag.TotalCount</strong> |
        Қоймада бар: <strong class="text-success">@ViewBag.AvailableCount</strong>
    </div>
</div>

@* Санат бойынша сүзгі (ViewBag арқылы келген тізім) *@
<div class="mb-4">
    <a asp-action="Index" class="btn btn-sm @(string.IsNullOrEmpty(selected) ? "btn-dark" : "btn-outline-dark") me-1 mb-1">Барлығы</a>
    @foreach (var c in categories)
    {
        <a asp-action="Index" asp-route-category="@c"
           class="btn btn-sm @(c == selected ? "btn-dark" : "btn-outline-dark") me-1 mb-1">@c</a>
    }
</div>

@if (!Model.Any())
{
    <div class="alert alert-info">Бұл санатта тауар жоқ.</div>
}
else
{
    <div class="row row-cols-1 row-cols-sm-2 row-cols-lg-3 row-cols-xl-4 g-4">
        @foreach (var product in Model)
        {
            <div class="col">
                <partial name="_ProductCard" model="product" />
            </div>
        }
    </div>
}
````

## `ProductCatalog/Views/Shared/Error.cshtml`

````razor

@model ErrorViewModel
@{
    ViewData["Title"] = "Қате";
}
<h1 class="text-danger">Қате орын алды</h1>
<p>Сұранысты өңдеу кезінде қате шықты.</p>
@if (Model.ShowRequestId)
{
    <p><strong>Request ID:</strong> <code>@Model.RequestId</code></p>
}
````

## `ProductCatalog/Views/Shared/_Layout.cshtml`

````razor

<!DOCTYPE html>
<html lang="kk">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] — ProductCatalog</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body class="d-flex flex-column min-vh-100">
    <header>
        <nav class="navbar navbar-expand-lg navbar-dark bg-dark shadow-sm">
            <div class="container">
                <a class="navbar-brand fw-bold" asp-controller="Product" asp-action="Index">
                    <i class="bi bi-shop"></i> ProductCatalog
                </a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#mainNav"
                        aria-controls="mainNav" aria-expanded="false" aria-label="Мәзір">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="mainNav">
                    <ul class="navbar-nav ms-auto">
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Product" asp-action="Index">Каталог</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Home" asp-action="About">Жоба туралы</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>

    <main role="main" class="container flex-grow-1 py-4">
        @RenderBody()
    </main>

    <footer class="bg-light border-top py-3 mt-auto">
        <div class="container text-center text-muted small">
            &copy; @DateTime.Now.Year — ProductCatalog | ASP.NET Core MVC, Razor, Bootstrap 5
        </div>
    </footer>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
````

## `ProductCatalog/Views/Shared/_ProductCard.cshtml`

````razor

@model ProductCatalog.Models.Product

@* Қайта қолданылатын тауар карточкасы (Partial View) *@
<div class="card h-100 product-card shadow-sm @(Model.IsAvailable ? "" : "opacity-75")">
    <div class="card-header d-flex justify-content-between align-items-center">
        <span class="badge bg-secondary">@Model.Category</span>
        <small class="text-muted">#@Model.Id</small>
    </div>
    <div class="card-body d-flex flex-column">
        <h5 class="card-title">@Model.Name</h5>
        <p class="card-text fs-4 fw-bold text-primary mb-3">@Model.Price.ToString("N0") ₸</p>

        @if (Model.IsAvailable)
        {
            <span class="badge bg-success align-self-start mb-3">
                <i class="bi bi-check-circle"></i> Қоймада бар
            </span>
        }
        else
        {
            <span class="badge bg-danger align-self-start mb-3">
                <i class="bi bi-x-circle"></i> Қоймада жоқ
            </span>
        }

        <a asp-controller="Product" asp-action="Details" asp-route-id="@Model.Id"
           class="btn btn-outline-primary mt-auto">Толығырақ</a>
    </div>
</div>
````

## `ProductCatalog/Views/_ViewImports.cshtml`

````razor

@using ProductCatalog
@using ProductCatalog.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
````

## `ProductCatalog/Views/_ViewStart.cshtml`

````razor

@{
    Layout = "_Layout";
}
````

## `ProductCatalog/wwwroot/css/site.css`

````css

html { font-size: 15px; }
@media (min-width: 768px) { html { font-size: 16px; } }

.product-card {
    transition: transform .15s ease, box-shadow .15s ease;
}
.product-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 .5rem 1rem rgba(0, 0, 0, .15) !important;
}
````

## `Lab02_StudentApp/Controllers/CourseController.cs`

````csharp

using Microsoft.AspNetCore.Mvc;
using StudentApp.Models;

namespace StudentApp.Controllers
{
    public class CourseController : Controller
    {
        // GET: /Course/List
        public IActionResult List()
        {
            var courses = new List<Course>
            {
                new Course { Id = 1, Title = "C# Web Apps", Credits = 5, Instructor = "Проф. Ахметов" },
                new Course { Id = 2, Title = "MS SQL Server", Credits = 4, Instructor = "Доц. Серіков" },
                new Course { Id = 3, Title = "Web Design (HTML/CSS)", Credits = 3, Instructor = "Аға оқыт. Жұмабаева" }
            };

            ViewData["Title"] = "Курстар тізімі";
            ViewBag.TotalCredits = courses.Sum(c => c.Credits);

            return View(courses);
        }
    }
}
````

## `Lab02_StudentApp/Controllers/StudentController.cs`

````csharp

using Microsoft.AspNetCore.Mvc;
using StudentApp.Models;

namespace StudentApp.Controllers
{
    public class StudentController : Controller
    {
        private static readonly List<Student> Students = new()
        {
            new Student { Id = 1, FullName = "Сейітқали Ерлан", Specialty = "Ақпараттық жүйелер", Gpa = 3.6 },
            new Student { Id = 2, FullName = "Нұрланова Әйгерім", Specialty = "Бағдарламалық инженерия", Gpa = 2.7 }
        };

        // GET: /Student/Details/1
        public IActionResult Details(int id = 1)
        {
            var student = Students.FirstOrDefault(s => s.Id == id);
            if (student == null) return NotFound();
            return View(student);
        }
    }
}
````

## `Lab02_StudentApp/Models/Course.cs`

````csharp

namespace StudentApp.Models
{
    public class Course
    {
        public int Id { get; set; }
        public string Title { get; set; } = string.Empty;
        public int Credits { get; set; }
        public string Instructor { get; set; } = string.Empty;
    }
}
````

## `Lab02_StudentApp/Models/Student.cs`

````csharp

namespace StudentApp.Models
{
    public class Student
    {
        public int Id { get; set; }
        public string FullName { get; set; } = string.Empty;
        public string Specialty { get; set; } = string.Empty;
        public double Gpa { get; set; }
    }
}
````

## `Lab02_StudentApp/Program.cs`

````csharp

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Course}/{action=List}/{id?}");

app.Run();
````

## `Lab02_StudentApp/StudentApp.csproj`

````xml

<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <RootNamespace>StudentApp</RootNamespace>
  </PropertyGroup>

</Project>
````

## `Lab02_StudentApp/Views/Course/List.cshtml`

````razor

@model List<StudentApp.Models.Course>

<h1 class="h2 mb-1">@ViewData["Title"]</h1>
<p class="text-muted mb-4">Жалпы кредит саны: <strong>@ViewBag.TotalCredits</strong></p>

<div class="row row-cols-1 row-cols-md-3 g-4">
    @foreach (var course in Model)
    {
        <div class="col">
            <partial name="_CourseCard" model="course" />
        </div>
    }
</div>
````

## `Lab02_StudentApp/Views/Shared/_CourseCard.cshtml`

````razor

@model StudentApp.Models.Course

<div class="card h-100 shadow-sm">
    <div class="card-body">
        <h5 class="card-title">@Model.Title</h5>
        <p class="card-text mb-1"><strong>Оқытушы:</strong> @Model.Instructor</p>
        <p class="card-text"><strong>Кредит:</strong> <span class="badge bg-info text-dark">@Model.Credits</span></p>
    </div>
    <div class="card-footer text-muted small">Курс ID: @Model.Id</div>
</div>
````

## `Lab02_StudentApp/Views/Shared/_Layout.cshtml`

````razor

<!DOCTYPE html>
<html lang="kk">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] — StudentApp</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
</head>
<body class="d-flex flex-column min-vh-100">
    <nav class="navbar navbar-expand navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" asp-controller="Course" asp-action="List">StudentApp</a>
            <div class="navbar-nav">
                <a class="nav-link" asp-controller="Course" asp-action="List">Курстар</a>
                <a class="nav-link" asp-controller="Student" asp-action="Details" asp-route-id="1">Студент профилі</a>
            </div>
        </div>
    </nav>
    <main class="container flex-grow-1 py-4">
        @RenderBody()
    </main>
    <footer class="border-top py-3 text-center text-muted small">
        &copy; @DateTime.Now.Year — StudentApp (2-апта зертханалық жұмысы)
    </footer>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
````

## `Lab02_StudentApp/Views/Student/Details.cshtml`

````razor

@model StudentApp.Models.Student
@{
    ViewData["Title"] = "Студент Профилі";
    Layout = "_Layout";
}
<div class="card my-3">
    <div class="card-header bg-primary text-white">
        <h3>@Model.FullName</h3>
    </div>
    <div class="card-body">
        <p><strong>ID:</strong> @Model.Id</p>
        <p><strong>Мамандығы:</strong> @Model.Specialty</p>
        <p><strong>GPA:</strong> @Model.Gpa</p>

        @if (Model.Gpa >= 3.0)
        {
            <span class="badge bg-success">Үздік студент</span>
        }
        else
        {
            <span class="badge bg-warning text-dark">Төмен GPA</span>
        }
    </div>
</div>
````

## `Lab02_StudentApp/Views/_ViewImports.cshtml`

````razor

@using StudentApp
@using StudentApp.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
````

## `Lab02_StudentApp/Views/_ViewStart.cshtml`

````razor

@{
    Layout = "_Layout";
}
````
