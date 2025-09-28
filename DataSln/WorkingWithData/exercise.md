# Event RSVP Management System - Step-by-Step Exercise

This exercise will guide you through building a complete Event RSVP Management System using **ASP.NET Core MVC** and **Entity Framework Core**. You'll implement CRUD operations, repository patterns, dependency injection, and unit testing.

## Prerequisites

- ✅ ASP.NET Core MVC project already created
- ✅ .NET 9.0 SDK installed
- ✅ Basic understanding of C# and MVC concepts

---

## Exercise Overview

You'll build this system step by step:

1. **[Step 1](#step-1-install-entity-framework-packages)** - Install EF Core packages
2. **[Step 2](#step-2-create-entity-models)** - Create Event and RSVP models
3. **[Step 3](#step-3-create-database-context)** - Set up DbContext
4. **[Step 4](#step-4-configure-database-connection)** - Configure SQLite connection
5. **[Step 5](#step-5-create-repository-interfaces)** - Define repository contracts
6. **[Step 6](#step-6-implement-repositories)** - Implement data access logic
7. **[Step 7](#step-7-configure-dependency-injection)** - Register services
8. **[Step 8](#step-8-create-database-and-seed-data)** - Add sample data
9. **[Step 9](#step-9-create-rsvp-controller)** - Build RSVP controller
10. **[Step 10](#step-10-update-home-controller)** - Display events
11. **[Step 11](#step-11-create-views)** - Build user interface
12. **[Step 12](#step-12-add-bootstrap-styling)** - Style with Bootstrap
13. **[Step 13](#step-13-add-navigation)** - Update layout and navigation
14. **[Step 14](#step-14-create-unit-tests)** - Write comprehensive tests
15. **[Step 15](#step-15-test-and-verify)** - Final testing and verification

---

## Step 1: Install Entity Framework Packages

First, add the necessary NuGet packages for Entity Framework Core and SQLite.

**Task**: Add EF Core packages to your project

```bash
# Navigate to your project directory
cd DataSln/WorkingWithData

# Add Entity Framework Core for SQLite
dotnet add package Microsoft.EntityFrameworkCore.Sqlite --version 9.0.0

# Add EF Core Design tools for migrations
dotnet add package Microsoft.EntityFrameworkCore.Design --version 9.0.0

# Add EF Core Tools for database operations
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 9.0.0
```

** Verification**: Check that packages are added to `WorkingWithData.csproj`

---

## Step 2: Create Entity Models

Create the data models that represent your domain entities.

** Task**: Create Event and RSVP entity classes

### 2.1 Create Event Model

Create `Models/Event.cs`:

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace WorkingWithData.Models
{
    public class Event
    {
        public int EventId { get; set; }

        [Required]
        [StringLength(100)]
        public string Name { get; set; } = string.Empty;

        [Required]
        [StringLength(500)]
        public string Description { get; set; } = string.Empty;

        [Required]
        public DateTime EventDate { get; set; }

        [Required]
        [StringLength(200)]
        public string Location { get; set; } = string.Empty;

        [Range(1, 1000)]
        public int MaxAttendees { get; set; }

        // Navigation property for related RSVPs
        public List<Rsvp> Rsvps { get; set; } = new List<Rsvp>();
    }
}
```

### 2.2 Create RSVP Model

Create `Models/Rsvp.cs`:

```csharp
using System.ComponentModel.DataAnnotations;

namespace WorkingWithData.Models
{
    public class Rsvp
    {
        public int RsvpId { get; set; }

        [Required]
        public int EventId { get; set; }

        [Required]
        [StringLength(100)]
        public string Name { get; set; } = string.Empty;

        [Required]
        [EmailAddress]
        [StringLength(200)]
        public string Email { get; set; } = string.Empty;

        [Phone]
        [StringLength(20)]
        public string Phone { get; set; } = string.Empty;

        public DateTime RsvpDate { get; set; } = DateTime.Now;

        // Navigation property to related Event
        public Event? Event { get; set; }
    }
}
```

** Verification**: Both model files compile without errors

---

## Step 3: Create Database Context

Create the Entity Framework DbContext to manage database operations.

** Task**: Create EventDbContext class

Create `Models/EventDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

namespace WorkingWithData.Models
{
    public class EventDbContext : DbContext
    {
        public EventDbContext(DbContextOptions<EventDbContext> options) : base(options)
        {
        }

        public DbSet<Event> Events => Set<Event>();
        public DbSet<Rsvp> Rsvps => Set<Rsvp>();

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Configure Event entity
            modelBuilder.Entity<Event>(entity =>
            {
                entity.HasKey(e => e.EventId);
                entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
                entity.Property(e => e.Description).IsRequired().HasMaxLength(500);
                entity.Property(e => e.Location).IsRequired().HasMaxLength(200);
            });

            // Configure RSVP entity
            modelBuilder.Entity<Rsvp>(entity =>
            {
                entity.HasKey(r => r.RsvpId);
                entity.Property(r => r.Name).IsRequired().HasMaxLength(100);
                entity.Property(r => r.Email).IsRequired().HasMaxLength(200);
                entity.Property(r => r.Phone).HasMaxLength(20);

                // Configure relationship
                entity.HasOne(r => r.Event)
                      .WithMany(e => e.Rsvps)
                      .HasForeignKey(r => r.EventId)
                      .OnDelete(DeleteBehavior.Cascade);
            });
        }
    }
}
```

** Verification**: DbContext compiles and includes both DbSets

---

## Step 4: Configure Database Connection

Set up the database connection string and configure EF Core in your application.

** Task**: Configure SQLite connection

### 4.1 Update appsettings.json

Add connection string to `appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=EventRsvp.db"
  }
}
```

### 4.2 Update Program.cs

Update `Program.cs` to configure EF Core:

```csharp
using Microsoft.EntityFrameworkCore;
using WorkingWithData.Models;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

// Add Entity Framework
builder.Services.AddDbContext<EventDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

** Verification**: Application starts without errors

---

## Step 5: Create Repository Interfaces

Implement the Repository pattern to abstract data access logic.

** Task**: Create repository interfaces

### 5.1 Create Repository Folder

Create folder: `Models/Repository/`

### 5.2 Create Event Repository Interface

Create `Models/Repository/IEventRepository.cs`:

```csharp
namespace WorkingWithData.Models.Repository
{
    public interface IEventRepository
    {
        IEnumerable<Event> GetAllEvents();
        Event? GetEventById(int id);
        void AddEvent(Event eventItem);
        void UpdateEvent(Event eventItem);
        void DeleteEvent(int id);
        void SaveChanges();
    }
}
```

### 5.3 Create RSVP Repository Interface

Create `Models/Repository/IRsvpRepository.cs`:

```csharp
namespace WorkingWithData.Models.Repository
{
    public interface IRsvpRepository
    {
        IEnumerable<Rsvp> GetAllRsvps();
        IEnumerable<Rsvp> GetRsvpsByEventId(int eventId);
        Rsvp? GetRsvpById(int id);
        void AddRsvp(Rsvp rsvp);
        void UpdateRsvp(Rsvp rsvp);
        void DeleteRsvp(int id);
        void SaveChanges();
    }
}
```

** Verification**: Both interfaces define clear contracts for data operations

---

## Step 6: Implement Repositories

Create concrete implementations of the repository interfaces.

** Task**: Implement repository classes

### 6.1 Create Event Repository Implementation

Create `Models/Repository/EventRepository.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

namespace WorkingWithData.Models.Repository
{
    public class EventRepository : IEventRepository
    {
        private readonly EventDbContext _context;

        public EventRepository(EventDbContext context)
        {
            _context = context;
        }

        public IEnumerable<Event> GetAllEvents()
        {
            return _context.Events
                          .Include(e => e.Rsvps)
                          .OrderBy(e => e.EventDate)
                          .ToList();
        }

        public Event? GetEventById(int id)
        {
            return _context.Events
                          .Include(e => e.Rsvps)
                          .FirstOrDefault(e => e.EventId == id);
        }

        public void AddEvent(Event eventItem)
        {
            _context.Events.Add(eventItem);
        }

        public void UpdateEvent(Event eventItem)
        {
            _context.Events.Update(eventItem);
        }

        public void DeleteEvent(int id)
        {
            var eventItem = _context.Events.Find(id);
            if (eventItem != null)
            {
                _context.Events.Remove(eventItem);
            }
        }

        public void SaveChanges()
        {
            _context.SaveChanges();
        }
    }
}
```

### 6.2 Create RSVP Repository Implementation

Create `Models/Repository/RsvpRepository.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

namespace WorkingWithData.Models.Repository
{
    public class RsvpRepository : IRsvpRepository
    {
        private readonly EventDbContext _context;

        public RsvpRepository(EventDbContext context)
        {
            _context = context;
        }

        public IEnumerable<Rsvp> GetAllRsvps()
        {
            return _context.Rsvps
                          .Include(r => r.Event)
                          .OrderBy(r => r.RsvpDate)
                          .ToList();
        }

        public IEnumerable<Rsvp> GetRsvpsByEventId(int eventId)
        {
            return _context.Rsvps
                          .Include(r => r.Event)
                          .Where(r => r.EventId == eventId)
                          .OrderBy(r => r.RsvpDate)
                          .ToList();
        }

        public Rsvp? GetRsvpById(int id)
        {
            return _context.Rsvps
                          .Include(r => r.Event)
                          .FirstOrDefault(r => r.RsvpId == id);
        }

        public void AddRsvp(Rsvp rsvp)
        {
            _context.Rsvps.Add(rsvp);
        }

        public void UpdateRsvp(Rsvp rsvp)
        {
            _context.Rsvps.Update(rsvp);
        }

        public void DeleteRsvp(int id)
        {
            var rsvp = _context.Rsvps.Find(id);
            if (rsvp != null)
            {
                _context.Rsvps.Remove(rsvp);
            }
        }

        public void SaveChanges()
        {
            _context.SaveChanges();
        }
    }
}
```

** Verification**: Repository implementations compile without errors

---

## Step 7: Configure Dependency Injection

Register your services in the dependency injection container.

** Task**: Update Program.cs to register repositories

Update `Program.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using WorkingWithData.Models;
using WorkingWithData.Models.Repository;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

// Add Entity Framework
builder.Services.AddDbContext<EventDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));

// Register repositories
builder.Services.AddScoped<IEventRepository, EventRepository>();
builder.Services.AddScoped<IRsvpRepository, RsvpRepository>();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

** Verification**: Application still compiles and runs

---

## Step 8: Create Database and Seed Data

Set up database migrations and seed initial data.

** Task**: Create migration and seed data

### 8.1 Create Initial Migration

```bash
# From the WorkingWithData project directory
dotnet ef migrations add InitialCreate
dotnet ef database update
```

### 8.2 Create Seed Data Class

Create `Data/SeedData.cs`:

```csharp
using WorkingWithData.Models;
using Microsoft.EntityFrameworkCore;

namespace WorkingWithData.Data
{
    public static class SeedData
    {
        public static void Initialize(IServiceProvider serviceProvider)
        {
            using var context = new EventDbContext(
                serviceProvider.GetRequiredService<DbContextOptions<EventDbContext>>());

            // Look for any events
            if (context.Events.Any())
            {
                return; // DB has been seeded
            }

            var events = new Event[]
            {
                new Event
                {
                    Name = "Tech Conference 2024",
                    Description = "Annual technology conference featuring the latest in software development, AI, and cloud computing.",
                    EventDate = DateTime.Now.AddDays(30),
                    Location = "Convention Center, Downtown",
                    MaxAttendees = 200
                },
                new Event
                {
                    Name = "Networking Night",
                    Description = "Professional networking event for local business professionals and entrepreneurs.",
                    EventDate = DateTime.Now.AddDays(15),
                    Location = "Grand Hotel Ballroom",
                    MaxAttendees = 150
                },
                new Event
                {
                    Name = "Coding Workshop",
                    Description = "Hands-on workshop covering modern web development frameworks and best practices.",
                    EventDate = DateTime.Now.AddDays(45),
                    Location = "Tech Hub Learning Center",
                    MaxAttendees = 50
                },
                new Event
                {
                    Name = "Startup Pitch Night",
                    Description = "Local entrepreneurs present their startup ideas to potential investors and mentors.",
                    EventDate = DateTime.Now.AddDays(20),
                    Location = "Innovation District",
                    MaxAttendees = 100
                },
                new Event
                {
                    Name = "Digital Marketing Seminar",
                    Description = "Learn the latest digital marketing strategies and social media trends.",
                    EventDate = DateTime.Now.AddDays(35),
                    Location = "Business Center Conference Room",
                    MaxAttendees = 75
                }
            };

            context.Events.AddRange(events);
            context.SaveChanges();
        }
    }
}
```

### 8.3 Update Program.cs to Call Seed Data

Update `Program.cs` to initialize seed data:

```csharp
using Microsoft.EntityFrameworkCore;
using WorkingWithData.Models;
using WorkingWithData.Models.Repository;
using WorkingWithData.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

// Add Entity Framework
builder.Services.AddDbContext<EventDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));

// Register repositories
builder.Services.AddScoped<IEventRepository, EventRepository>();
builder.Services.AddScoped<IRsvpRepository, RsvpRepository>();

var app = builder.Build();

// Seed the database
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    SeedData.Initialize(services);
}

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

** Verification**: Run the app and check that EventRsvp.db is created and seeded

---

## Step 9: Create RSVP Controller

Build the controller to handle RSVP CRUD operations.

** Task**: Create RsvpController

Create `Controllers/RsvpController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using WorkingWithData.Models;
using WorkingWithData.Models.Repository;

namespace WorkingWithData.Controllers
{
    public class RsvpController : Controller
    {
        private readonly IRsvpRepository _rsvpRepository;
        private readonly IEventRepository _eventRepository;

        public RsvpController(IRsvpRepository rsvpRepository, IEventRepository eventRepository)
        {
            _rsvpRepository = rsvpRepository;
            _eventRepository = eventRepository;
        }

        // GET: Rsvp
        public IActionResult Index()
        {
            var rsvps = _rsvpRepository.GetAllRsvps();
            return View(rsvps);
        }

        // GET: Rsvp/Details/5
        public IActionResult Details(int id)
        {
            var rsvp = _rsvpRepository.GetRsvpById(id);
            if (rsvp == null)
            {
                return NotFound();
            }
            return View(rsvp);
        }

        // GET: Rsvp/Create/5 (5 is EventId)
        public IActionResult Create(int eventId)
        {
            var eventItem = _eventRepository.GetEventById(eventId);
            if (eventItem == null)
            {
                return NotFound();
            }

            var rsvp = new Rsvp { EventId = eventId, Event = eventItem };
            return View(rsvp);
        }

        // POST: Rsvp/Create
        [HttpPost]
        [ValidateAntiForgeryToken]
        public IActionResult Create(Rsvp rsvp)
        {
            if (ModelState.IsValid)
            {
                rsvp.RsvpDate = DateTime.Now;
                _rsvpRepository.AddRsvp(rsvp);
                _rsvpRepository.SaveChanges();
                return RedirectToAction(nameof(Index));
            }

            // Reload event if validation fails
            rsvp.Event = _eventRepository.GetEventById(rsvp.EventId);
            return View(rsvp);
        }

        // GET: Rsvp/Edit/5
        public IActionResult Edit(int id)
        {
            var rsvp = _rsvpRepository.GetRsvpById(id);
            if (rsvp == null)
            {
                return NotFound();
            }
            return View(rsvp);
        }

        // POST: Rsvp/Edit/5
        [HttpPost]
        [ValidateAntiForgeryToken]
        public IActionResult Edit(int id, Rsvp rsvp)
        {
            if (id != rsvp.RsvpId)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                _rsvpRepository.UpdateRsvp(rsvp);
                _rsvpRepository.SaveChanges();
                return RedirectToAction(nameof(Index));
            }

            // Reload event if validation fails
            rsvp.Event = _eventRepository.GetEventById(rsvp.EventId);
            return View(rsvp);
        }

        // GET: Rsvp/Delete/5
        public IActionResult Delete(int id)
        {
            var rsvp = _rsvpRepository.GetRsvpById(id);
            if (rsvp == null)
            {
                return NotFound();
            }
            return View(rsvp);
        }

        // POST: Rsvp/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public IActionResult DeleteConfirmed(int id)
        {
            _rsvpRepository.DeleteRsvp(id);
            _rsvpRepository.SaveChanges();
            return RedirectToAction(nameof(Index));
        }
    }
}
```

** Verification**: Controller compiles without errors

---

## Step 10: Update Home Controller

Update the Home controller to display events.

** Task**: Update HomeController to show events

Update `Controllers/HomeController.cs`:

```csharp
using System.Diagnostics;
using Microsoft.AspNetCore.Mvc;
using WorkingWithData.Models;
using WorkingWithData.Models.Repository;

namespace WorkingWithData.Controllers
{
    public class HomeController : Controller
    {
        private readonly ILogger<HomeController> _logger;
        private readonly IEventRepository _eventRepository;

        public HomeController(ILogger<HomeController> logger, IEventRepository eventRepository)
        {
            _logger = logger;
            _eventRepository = eventRepository;
        }

        public IActionResult Index()
        {
            var events = _eventRepository.GetAllEvents();
            return View(events);
        }

        public IActionResult Privacy()
        {
            return View();
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
            return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
        }
    }
}
```

** Verification**: Home controller compiles without errors

---

## Step 11: Create Views

Build the user interface views for the application.

** Task**: Create all necessary views

### 11.1 Update Home Index View

Update `Views/Home/Index.cshtml`:

```html
@model IEnumerable<WorkingWithData.Models.Event>

@{
    ViewData["Title"] = "Upcoming Events";
}

<div class="container">
    <div class="row">
        <div class="col-12">
            <h1 class="display-4 text-center mb-4">🎉 Upcoming Events</h1>
            <p class="lead text-center text-muted mb-5">Discover exciting events and RSVP today!</p>
        </div>
    </div>

    <div class="row">
        @foreach (var eventItem in Model)
        {
            <div class="col-md-6 col-lg-4 mb-4">
                <div class="card h-100 shadow-sm">
                    <div class="card-body">
                        <h5 class="card-title">@eventItem.Name</h5>
                        <p class="card-text">@eventItem.Description</p>
                        <ul class="list-unstyled">
                            <li><strong>📅 Date:</strong> @eventItem.EventDate.ToString("MMM dd, yyyy - h:mm tt")</li>
                            <li><strong>📍 Location:</strong> @eventItem.Location</li>
                            <li><strong>👥 Max Attendees:</strong> @eventItem.MaxAttendees</li>
                            <li><strong>✅ Current RSVPs:</strong> @eventItem.Rsvps.Count</li>
                        </ul>
                    </div>
                    <div class="card-footer bg-transparent">
                        <a href="@Url.Action("Create", "Rsvp", new { eventId = eventItem.EventId })"
                           class="btn btn-primary w-100">
                            RSVP Now
                        </a>
                    </div>
                </div>
            </div>
        }
    </div>

    <div class="row mt-4">
        <div class="col-12 text-center">
            <a href="@Url.Action("Index", "Rsvp")" class="btn btn-outline-secondary">
                View All RSVPs
            </a>
        </div>
    </div>
</div>
```

### 11.2 Create RSVP Views Folder

Create folder: `Views/Rsvp/`

### 11.3 Create RSVP Index View

Create `Views/Rsvp/Index.cshtml`:

```html
@model IEnumerable<WorkingWithData.Models.Rsvp>

@{
    ViewData["Title"] = "All RSVPs";
}

<div class="container">
    <h1> All RSVPs</h1>

    <div class="row mb-3">
        <div class="col">
            <a href="@Url.Action("Index", "Home")" class="btn btn-outline-primary">
                ← Back to Events
            </a>
        </div>
    </div>

    @if (Model.Any())
    {
        <div class="table-responsive">
            <table class="table table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>Event</th>
                        <th>Name</th>
                        <th>Email</th>
                        <th>Phone</th>
                        <th>RSVP Date</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach (var rsvp in Model)
                    {
                        <tr>
                            <td>@rsvp.Event?.Name</td>
                            <td>@rsvp.Name</td>
                            <td>@rsvp.Email</td>
                            <td>@rsvp.Phone</td>
                            <td>@rsvp.RsvpDate.ToString("MMM dd, yyyy")</td>
                            <td>
                                <a href="@Url.Action("Details", new { id = rsvp.RsvpId })"
                                   class="btn btn-sm btn-outline-info">Details</a>
                                <a href="@Url.Action("Edit", new { id = rsvp.RsvpId })"
                                   class="btn btn-sm btn-outline-warning">Edit</a>
                                <a href="@Url.Action("Delete", new { id = rsvp.RsvpId })"
                                   class="btn btn-sm btn-outline-danger">Delete</a>
                            </td>
                        </tr>
                    }
                </tbody>
            </table>
        </div>
    }
    else
    {
        <div class="alert alert-info">
            <h4>No RSVPs yet!</h4>
            <p>Be the first to RSVP to an event.</p>
            <a href="@Url.Action("Index", "Home")" class="btn btn-primary">View Events</a>
        </div>
    }
</div>
```

### 11.4 Create RSVP Create View

Create `Views/Rsvp/Create.cshtml`:

```html
@model WorkingWithData.Models.Rsvp

@{
    ViewData["Title"] = "RSVP to Event";
}

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3>🎉 RSVP to: @Model.Event?.Name</h3>
                </div>
                <div class="card-body">
                    @if (Model.Event != null)
                    {
                        <div class="alert alert-info">
                            <h5>Event Details:</h5>
                            <p><strong>Date:</strong> @Model.Event.EventDate.ToString("MMM dd, yyyy - h:mm tt")</p>
                            <p><strong>Location:</strong> @Model.Event.Location</p>
                            <p><strong>Description:</strong> @Model.Event.Description</p>
                        </div>
                    }

                    <form asp-action="Create" method="post">
                        <input type="hidden" asp-for="EventId" />

                        <div class="mb-3">
                            <label asp-for="Name" class="form-label">Full Name *</label>
                            <input asp-for="Name" class="form-control" placeholder="Enter your full name" />
                            <span asp-validation-for="Name" class="text-danger"></span>
                        </div>

                        <div class="mb-3">
                            <label asp-for="Email" class="form-label">Email Address *</label>
                            <input asp-for="Email" class="form-control" placeholder="Enter your email address" />
                            <span asp-validation-for="Email" class="text-danger"></span>
                        </div>

                        <div class="mb-3">
                            <label asp-for="Phone" class="form-label">Phone Number</label>
                            <input asp-for="Phone" class="form-control" placeholder="Enter your phone number" />
                            <span asp-validation-for="Phone" class="text-danger"></span>
                        </div>

                        <div class="d-grid gap-2">
                            <button type="submit" class="btn btn-success btn-lg">
                                 Confirm RSVP
                            </button>
                            <a href="@Url.Action("Index", "Home")" class="btn btn-outline-secondary">
                                Cancel
                            </a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
```

### 11.5 Create RSVP Edit View

Create `Views/Rsvp/Edit.cshtml`:

```html
@model WorkingWithData.Models.Rsvp

@{
    ViewData["Title"] = "Edit RSVP";
}

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3>Edit RSVP</h3>
                </div>
                <div class="card-body">
                    <form asp-action="Edit" method="post">
                        <input type="hidden" asp-for="RsvpId" />
                        <input type="hidden" asp-for="EventId" />
                        <input type="hidden" asp-for="RsvpDate" />

                        <div class="mb-3">
                            <label asp-for="Name" class="form-label">Full Name *</label>
                            <input asp-for="Name" class="form-control" />
                            <span asp-validation-for="Name" class="text-danger"></span>
                        </div>

                        <div class="mb-3">
                            <label asp-for="Email" class="form-label">Email Address *</label>
                            <input asp-for="Email" class="form-control" />
                            <span asp-validation-for="Email" class="text-danger"></span>
                        </div>

                        <div class="mb-3">
                            <label asp-for="Phone" class="form-label">Phone Number</label>
                            <input asp-for="Phone" class="form-control" />
                            <span asp-validation-for="Phone" class="text-danger"></span>
                        </div>

                        <div class="d-grid gap-2">
                            <button type="submit" class="btn btn-warning btn-lg">
                                Update RSVP
                            </button>
                            <a href="@Url.Action("Index")" class="btn btn-outline-secondary">
                                Cancel
                            </a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
```

### 11.6 Create RSVP Details View

Create `Views/Rsvp/Details.cshtml`:

```html
@model WorkingWithData.Models.Rsvp

@{
    ViewData["Title"] = "RSVP Details";
}

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3>RSVP Details</h3>
                </div>
                <div class="card-body">
                    <dl class="row">
                        <dt class="col-sm-3">Event:</dt>
                        <dd class="col-sm-9">@Model.Event?.Name</dd>

                        <dt class="col-sm-3">Name:</dt>
                        <dd class="col-sm-9">@Model.Name</dd>

                        <dt class="col-sm-3">Email:</dt>
                        <dd class="col-sm-9">@Model.Email</dd>

                        <dt class="col-sm-3">Phone:</dt>
                        <dd class="col-sm-9">@Model.Phone</dd>

                        <dt class="col-sm-3">RSVP Date:</dt>
                        <dd class="col-sm-9">@Model.RsvpDate.ToString("MMM dd, yyyy - h:mm tt")</dd>
                    </dl>

                    <div class="mt-3">
                        <a href="@Url.Action("Edit", new { id = Model.RsvpId })" class="btn btn-warning">
                             Edit
                        </a>
                        <a href="@Url.Action("Index")" class="btn btn-outline-secondary">
                            ← Back to List
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

### 11.7 Create RSVP Delete View

Create `Views/Rsvp/Delete.cshtml`:

```html
@model WorkingWithData.Models.Rsvp

@{
    ViewData["Title"] = "Delete RSVP";
}

<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card border-danger">
                <div class="card-header bg-danger text-white">
                    <h3>Delete RSVP</h3>
                </div>
                <div class="card-body">
                    <div class="alert alert-warning">
                        <h4>Are you sure you want to delete this RSVP?</h4>
                        <p>This action cannot be undone.</p>
                    </div>

                    <dl class="row">
                        <dt class="col-sm-3">Event:</dt>
                        <dd class="col-sm-9">@Model.Event?.Name</dd>

                        <dt class="col-sm-3">Name:</dt>
                        <dd class="col-sm-9">@Model.Name</dd>

                        <dt class="col-sm-3">Email:</dt>
                        <dd class="col-sm-9">@Model.Email</dd>

                        <dt class="col-sm-3">RSVP Date:</dt>
                        <dd class="col-sm-9">@Model.RsvpDate.ToString("MMM dd, yyyy")</dd>
                    </dl>

                    <form asp-action="Delete" method="post" class="d-inline">
                        <input type="hidden" asp-for="RsvpId" />
                        <button type="submit" class="btn btn-danger">
                            Yes, Delete
                        </button>
                    </form>

                    <a href="@Url.Action("Index")" class="btn btn-outline-secondary">
                        Cancel
                    </a>
                </div>
            </div>
        </div>
    </div>
</div>
```

**Verification**: All views are created and use proper Bootstrap styling

---

## Step 12: Add Bootstrap Styling

Ensure your layout uses Bootstrap 5 and add custom styles.

**Task**: Update layout and add custom CSS

### 12.1 Update Layout File

Update `Views/Shared/_Layout.cshtml`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Event RSVP System</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/WorkingWithData.styles.css" asp-append-version="true" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container-fluid">
                <a class="navbar-brand" href="@Url.Action("Index", "Home")">Event RSVP</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse"
                        aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="@Url.Action("Index", "Home")">Events</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="@Url.Action("Index", "Rsvp")">RSVPs</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" href="@Url.Action("Privacy", "Home")">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>

    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2024 - Event RSVP System - <a href="@Url.Action("Privacy", "Home")">Privacy</a>
        </div>
    </footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

### 12.2 Update Site CSS

Update `wwwroot/css/site.css`:

```css
/* Custom styles for Event RSVP System */

html {
  font-size: 14px;
}

@media (min-width: 768px) {
  html {
    font-size: 16px;
  }
}

.btn:focus,
.btn:active:focus,
.btn-link.nav-link:focus,
.form-control:focus,
.form-check-input:focus {
  box-shadow: 0 0 0 0.1rem white, 0 0 0 0.25rem #258cfb;
}

html {
  position: relative;
  min-height: 100%;
}

body {
  margin-bottom: 60px;
  background-color: #f8f9fa;
}

.footer {
  position: absolute;
  bottom: 0;
  width: 100%;
  white-space: nowrap;
  line-height: 60px;
  background-color: #fff;
}

/* Custom card styling */
.card {
  transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
  border: none;
  border-radius: 12px;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15) !important;
}

.card-title {
  color: #2c3e50;
  font-weight: 600;
}

.card-text {
  color: #6c757d;
  line-height: 1.6;
}

/* Event card specific styling */
.card-footer {
  border-top: 1px solid rgba(0, 0, 0, 0.08);
  padding: 1rem;
}

/* Navigation styling */
.navbar-brand {
  font-weight: 700;
  font-size: 1.4rem;
}

.nav-link {
  font-weight: 500;
}

/* Button enhancements */
.btn {
  border-radius: 8px;
  font-weight: 500;
  padding: 0.5rem 1.5rem;
}

.btn-lg {
  padding: 0.75rem 2rem;
  font-size: 1.1rem;
}

/* Form styling */
.form-control {
  border-radius: 8px;
  border: 2px solid #e9ecef;
  padding: 0.75rem 1rem;
}

.form-control:focus {
  border-color: #80bdff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

/* Alert styling */
.alert {
  border-radius: 8px;
  border: none;
}

/* Table styling */
.table {
  background-color: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.table-responsive {
  border-radius: 8px;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .display-4 {
    font-size: 2rem;
  }

  .card {
    margin-bottom: 1rem;
  }
}
```

**Verification**: Application has modern, attractive styling

---

## Step 13: Add Navigation

Update your layout to include proper navigation between sections.

**Task**: Verify navigation is working properly

The navigation should already be set up in your layout. Test these links:

- Events page (Home/Index)
- RSVPs page (Rsvp/Index)
- Privacy page
- RSVP creation from event cards
- RSVP editing and deletion

**Verification**: All navigation works smoothly between pages

---

## Step 14: Create Unit Tests

Write comprehensive unit tests for your repositories and controllers.

**Task**: Create unit tests

### 14.1 Add Moq Package to Test Project

```bash
# Navigate to test project directory
cd ../WorkingWithData.Tests

# Add Moq package
dotnet add package Moq --version 4.20.69

# Add Entity Framework In-Memory package for testing
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 9.0.0
```

### 14.2 Create Repository Tests

Update `UnitTest1.cs` or create `RepositoryTests.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using WorkingWithData.Models;
using WorkingWithData.Models.Repository;

namespace WorkingWithData.Tests
{
    public class RepositoryTests : IDisposable
    {
        private readonly EventDbContext _context;
        private readonly EventRepository _eventRepository;
        private readonly RsvpRepository _rsvpRepository;

        public RepositoryTests()
        {
            var options = new DbContextOptionsBuilder<EventDbContext>()
                .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
                .Options;

            _context = new EventDbContext(options);
            _eventRepository = new EventRepository(_context);
            _rsvpRepository = new RsvpRepository(_context);

            SeedTestData();
        }

        private void SeedTestData()
        {
            var events = new[]
            {
                new Event
                {
                    EventId = 1,
                    Name = "Test Event 1",
                    Description = "Test Description 1",
                    EventDate = DateTime.Now.AddDays(30),
                    Location = "Test Location 1",
                    MaxAttendees = 100
                },
                new Event
                {
                    EventId = 2,
                    Name = "Test Event 2",
                    Description = "Test Description 2",
                    EventDate = DateTime.Now.AddDays(45),
                    Location = "Test Location 2",
                    MaxAttendees = 150
                }
            };

            _context.Events.AddRange(events);
            _context.SaveChanges();
        }

        [Fact]
        public void GetAllEvents_ReturnsAllEvents()
        {
            // Act
            var result = _eventRepository.GetAllEvents();

            // Assert
            Assert.Equal(2, result.Count());
        }

        [Fact]
        public void GetEventById_ValidId_ReturnsEvent()
        {
            // Act
            var result = _eventRepository.GetEventById(1);

            // Assert
            Assert.NotNull(result);
            Assert.Equal("Test Event 1", result.Name);
        }

        [Fact]
        public void GetEventById_InvalidId_ReturnsNull()
        {
            // Act
            var result = _eventRepository.GetEventById(999);

            // Assert
            Assert.Null(result);
        }

        [Fact]
        public void AddEvent_ValidEvent_AddsToDatabase()
        {
            // Arrange
            var newEvent = new Event
            {
                Name = "New Test Event",
                Description = "New Test Description",
                EventDate = DateTime.Now.AddDays(60),
                Location = "New Test Location",
                MaxAttendees = 75
            };

            // Act
            _eventRepository.AddEvent(newEvent);
            _eventRepository.SaveChanges();

            // Assert
            var allEvents = _eventRepository.GetAllEvents();
            Assert.Equal(3, allEvents.Count());
            Assert.Contains(allEvents, e => e.Name == "New Test Event");
        }

        [Fact]
        public void AddRsvp_ValidRsvp_AddsToDatabase()
        {
            // Arrange
            var newRsvp = new Rsvp
            {
                EventId = 1,
                Name = "John Doe",
                Email = "john@example.com",
                Phone = "123-456-7890",
                RsvpDate = DateTime.Now
            };

            // Act
            _rsvpRepository.AddRsvp(newRsvp);
            _rsvpRepository.SaveChanges();

            // Assert
            var allRsvps = _rsvpRepository.GetAllRsvps();
            Assert.Single(allRsvps);
            Assert.Equal("John Doe", allRsvps.First().Name);
        }

        [Fact]
        public void GetRsvpsByEventId_ValidEventId_ReturnsCorrectRsvps()
        {
            // Arrange
            var rsvp1 = new Rsvp { EventId = 1, Name = "Person 1", Email = "person1@example.com" };
            var rsvp2 = new Rsvp { EventId = 1, Name = "Person 2", Email = "person2@example.com" };
            var rsvp3 = new Rsvp { EventId = 2, Name = "Person 3", Email = "person3@example.com" };

            _context.Rsvps.AddRange(rsvp1, rsvp2, rsvp3);
            _context.SaveChanges();

            // Act
            var result = _rsvpRepository.GetRsvpsByEventId(1);

            // Assert
            Assert.Equal(2, result.Count());
            Assert.All(result, r => Assert.Equal(1, r.EventId));
        }

        [Fact]
        public void DeleteRsvp_ValidId_RemovesFromDatabase()
        {
            // Arrange
            var rsvp = new Rsvp
            {
                EventId = 1,
                Name = "Test Person",
                Email = "test@example.com"
            };
            _context.Rsvps.Add(rsvp);
            _context.SaveChanges();

            var rsvpId = rsvp.RsvpId;

            // Act
            _rsvpRepository.DeleteRsvp(rsvpId);
            _rsvpRepository.SaveChanges();

            // Assert
            var deletedRsvp = _rsvpRepository.GetRsvpById(rsvpId);
            Assert.Null(deletedRsvp);
        }

        public void Dispose()
        {
            _context.Dispose();
        }
    }
}
```

### 14.3 Create Controller Tests

Create `ControllerTests.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using Moq;
using WorkingWithData.Controllers;
using WorkingWithData.Models;
using WorkingWithData.Models.Repository;

namespace WorkingWithData.Tests
{
    public class ControllerTests
    {
        private readonly Mock<IEventRepository> _mockEventRepository;
        private readonly Mock<IRsvpRepository> _mockRsvpRepository;
        private readonly Mock<ILogger<HomeController>> _mockLogger;

        public ControllerTests()
        {
            _mockEventRepository = new Mock<IEventRepository>();
            _mockRsvpRepository = new Mock<IRsvpRepository>();
            _mockLogger = new Mock<ILogger<HomeController>>();
        }

        [Fact]
        public void HomeController_Index_ReturnsViewWithEvents()
        {
            // Arrange
            var events = new List<Event>
            {
                new Event { EventId = 1, Name = "Event 1" },
                new Event { EventId = 2, Name = "Event 2" }
            };

            _mockEventRepository.Setup(repo => repo.GetAllEvents())
                               .Returns(events);

            var controller = new HomeController(_mockLogger.Object, _mockEventRepository.Object);

            // Act
            var result = controller.Index();

            // Assert
            var viewResult = Assert.IsType<ViewResult>(result);
            var model = Assert.IsAssignableFrom<IEnumerable<Event>>(viewResult.ViewData.Model);
            Assert.Equal(2, model.Count());
        }

        [Fact]
        public void RsvpController_Create_Get_ReturnsViewWithEvent()
        {
            // Arrange
            var eventItem = new Event
            {
                EventId = 1,
                Name = "Test Event",
                Description = "Test Description",
                EventDate = DateTime.Now.AddDays(30),
                Location = "Test Location",
                MaxAttendees = 100
            };

            _mockEventRepository.Setup(repo => repo.GetEventById(1))
                               .Returns(eventItem);

            var controller = new RsvpController(_mockRsvpRepository.Object, _mockEventRepository.Object);

            // Act
            var result = controller.Create(1);

            // Assert
            var viewResult = Assert.IsType<ViewResult>(result);
            var model = Assert.IsType<Rsvp>(viewResult.ViewData.Model);
            Assert.Equal(1, model.EventId);
            Assert.Equal("Test Event", model.Event?.Name);
        }

        [Fact]
        public void RsvpController_Create_Post_ValidModel_RedirectsToIndex()
        {
            // Arrange
            var rsvp = new Rsvp
            {
                EventId = 1,
                Name = "John Doe",
                Email = "john@example.com",
                Phone = "123-456-7890"
            };

            var controller = new RsvpController(_mockRsvpRepository.Object, _mockEventRepository.Object);

            // Act
            var result = controller.Create(rsvp);

            // Assert
            var redirectResult = Assert.IsType<RedirectToActionResult>(result);
            Assert.Equal("Index", redirectResult.ActionName);

            // Verify repository methods were called
            _mockRsvpRepository.Verify(repo => repo.AddRsvp(It.IsAny<Rsvp>()), Times.Once);
            _mockRsvpRepository.Verify(repo => repo.SaveChanges(), Times.Once);
        }

        [Fact]
        public void RsvpController_Delete_Get_ValidId_ReturnsViewWithRsvp()
        {
            // Arrange
            var rsvp = new Rsvp
            {
                RsvpId = 1,
                EventId = 1,
                Name = "John Doe",
                Email = "john@example.com",
                Event = new Event { EventId = 1, Name = "Test Event" }
            };

            _mockRsvpRepository.Setup(repo => repo.GetRsvpById(1))
                              .Returns(rsvp);

            var controller = new RsvpController(_mockRsvpRepository.Object, _mockEventRepository.Object);

            // Act
            var result = controller.Delete(1);

            // Assert
            var viewResult = Assert.IsType<ViewResult>(result);
            var model = Assert.IsType<Rsvp>(viewResult.ViewData.Model);
            Assert.Equal("John Doe", model.Name);
        }

        [Fact]
        public void RsvpController_Delete_Get_InvalidId_ReturnsNotFound()
        {
            // Arrange
            _mockRsvpRepository.Setup(repo => repo.GetRsvpById(999))
                              .Returns((Rsvp?)null);

            var controller = new RsvpController(_mockRsvpRepository.Object, _mockEventRepository.Object);

            // Act
            var result = controller.Delete(999);

            // Assert
            Assert.IsType<NotFoundResult>(result);
        }
    }
}
```

### 14.4 Run Tests

```bash
# Run all tests
dotnet test

# Run tests with detailed output
dotnet test --verbosity normal
```

**Verification**: All tests pass successfully

---

## Step 15: Test and Verify

Perform final testing and verification of your application.

**Task**: Complete end-to-end testing

### 15.1 Run the Application

```bash
# Navigate back to main project
cd ../WorkingWithData

# Run the application
dotnet run
```

### 15.2 Test All Functionality

1. **Home Page**:

   - ✅ Should display 5 seeded events in card layout
   - ✅ Each card shows event details (name, date, location, attendees)
   - ✅ RSVP buttons work and navigate correctly

2. **RSVP Creation**:

   - ✅ Form loads with event details
   - ✅ Validation works for required fields
   - ✅ Email validation functions properly
   - ✅ Successfully creates RSVP and redirects

3. **RSVP Management**:

   - ✅ RSVP list shows all created RSVPs
   - ✅ Edit functionality works
   - ✅ Delete confirmation works
   - ✅ Details view displays correctly

4. **Navigation**:

   - ✅ All navigation links work
   - ✅ Back buttons function properly
   - ✅ Breadcrumb navigation is intuitive

5. **Database**:
   - ✅ SQLite database is created
   - ✅ Data persists between sessions
   - ✅ Relationships work correctly

### 15.3 Verify Architecture

1. **Repository Pattern**: ✅ Data access is abstracted
2. **Dependency Injection**: ✅ Services are properly injected
3. **Entity Framework**: ✅ CRUD operations work
4. **Testing**: ✅ Unit tests cover key functionality
5. **UI/UX**: ✅ Bootstrap styling is applied and responsive

**✅ Verification**: Complete application works as expected

---

##Congratulations!

You have successfully built a complete Event RSVP Management System that demonstrates:

### ✅ What You've Learned

1. **Entity Framework Core**:

   - DbContext setup and configuration
   - Entity models with data annotations
   - Database relationships (one-to-many)
   - SQLite integration
   - Database seeding

2. **Repository Pattern**:

   - Interface-based design
   - Separation of concerns
   - Testable architecture
   - CRUD operations abstraction

3. **ASP.NET Core MVC**:

   - Controllers with dependency injection
   - Action methods for CRUD operations
   - Model validation
   - Routing and navigation

4. **User Interface**:

   - Responsive design with Bootstrap 5
   - Modern card-based layouts
   - Form handling and validation
   - Intuitive user experience

5. **Testing**:
   - Unit testing with xUnit
   - Mocking with Moq
   - In-memory database testing
   - Controller and repository testing

### Next Steps

Now that you've completed this exercise, consider extending the project with:

- **Authentication**: Add user login/registration
- **Authorization**: Different user roles (admin, user)
- **Email Notifications**: Send RSVP confirmations
- **Calendar Integration**: Export events to calendar
- **Advanced Filtering**: Search and filter events
- **API Endpoints**: RESTful API for mobile apps

### Additional Learning Resources

- [Entity Framework Core Documentation](https://docs.microsoft.com/en-us/ef/core/)
- [ASP.NET Core MVC Documentation](https://docs.microsoft.com/en-us/aspnet/core/mvc/)
- [Unit Testing in .NET](https://docs.microsoft.com/en-us/dotnet/core/testing/)
- [Bootstrap 5 Documentation](https://getbootstrap.com/docs/5.0/)
