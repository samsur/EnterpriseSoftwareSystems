# Event RSVP Management System

An educational **ASP.NET Core MVC** project that demonstrates **Entity Framework Core** fundamentals using a simple Event RSVP management system with **SQLite database**.

## Project Overview

This project teaches essential Entity Framework Core concepts through a practical Event RSVP system where users can:

- View upcoming events in an attractive card layout
- RSVP to events with their contact information
- Update their RSVP details
- Cancel their RSVP
- See a list of all RSVPs for each event

## Learning Objectives

By completing this project, you will learn:

### 🗄️ Entity Framework Core

- ✅ Creating and configuring `DbContext`
- ✅ Defining entity models with data annotations
- ✅ Setting up SQLite database connection
- ✅ Database seeding for initial data
- ✅ CRUD operations (Create, Read, Update, Delete)

### Architecture Patterns

- ✅ Repository pattern implementation
- ✅ Dependency injection in ASP.NET Core
- ✅ Separation of concerns

### Testing

- ✅ Unit testing with xUnit
- ✅ Mocking with Moq framework
- ✅ Testing repository patterns
- ✅ Testing controllers with dependency injection

### UI Development

- ✅ Bootstrap 5 for responsive design
- ✅ Card-based layouts for modern UI
- ✅ Form handling and validation
- ✅ Simple, clean interface design

## Features

### Event Management

- **Event Display**: Beautiful card layout showing event details
- **Event Seeding**: 5 sample events pre-loaded in the database

### RSVP System

- **Create RSVP**: Simple form to RSVP with name, email, and phone
- **View RSVPs**: List all RSVPs for events
- **Update RSVP**: Modify existing RSVP details
- **Delete RSVP**: Cancel/remove RSVP entries

### Data Models

- **Event**: ID, Name, Description, Date, Location, MaxAttendees
- **RSVP**: ID, EventID, Name, Email, Phone, RSVPDate

## Technical Stack

- **Framework**: ASP.NET Core 9.0 MVC
- **Database**: SQLite with Entity Framework Core
- **Testing**: xUnit + Moq
- **UI**: Bootstrap 5
- **Architecture**: Repository Pattern + Dependency Injection

## Project Structure

```
WorkingWithData/
├── Controllers/
│   ├── HomeController.cs          # Main events listing
│   └── RsvpController.cs          # RSVP CRUD operations
├── Models/
│   ├── Event.cs                   # Event entity model
│   ├── Rsvp.cs                    # RSVP entity model
│   ├── EventDbContext.cs          # Database context
│   └── Repository/
│       ├── IEventRepository.cs    # Event repository interface
│       ├── EventRepository.cs     # Event repository implementation
│       ├── IRsvpRepository.cs     # RSVP repository interface
│       └── RsvpRepository.cs      # RSVP repository implementation
├── Views/
│   ├── Home/
│   │   └── Index.cshtml          # Events card layout
│   ├── Rsvp/
│   │   ├── Index.cshtml          # RSVP list view
│   │   ├── Create.cshtml         # RSVP form
│   │   ├── Edit.cshtml           # Edit RSVP form
│   │   └── Details.cshtml        # RSVP details view
│   └── Shared/
│       └── _Layout.cshtml        # Bootstrap layout
└── Data/
    └── SeedData.cs               # Database seeding
```

## UI Design

The interface uses **Bootstrap 5** for a modern, responsive design:

- **Home Page**: Clean card layout displaying events with RSVP buttons
- **RSVP Forms**: Simple, intuitive forms with validation
- **RSVP Management**: Easy-to-use list view with edit/delete actions
- **Navigation**: Clean bootstrap navbar with logical flow

## Key Learning Points

### 1. Entity Framework Setup

Learn how to configure EF Core with SQLite, including connection strings and dependency injection.

### 2. Repository Pattern

Understand how to abstract data access logic and make it testable through interfaces.

### 3. CRUD Operations

Implement all four basic database operations in a real-world context.

### 4. Data Relationships

Work with related entities (Events and RSVPs) and foreign key relationships.

### 5. Testing Strategy

Write comprehensive unit tests for both repository classes and controllers.

## Getting Started

1. **Prerequisites**: .NET 9.0 SDK, Visual Studio 2022/VS Code
2. **Database**: SQLite database will be created automatically
3. **Dependencies**: Entity Framework Core, Bootstrap 5 (included)
4. **Run Project**: `dotnet run` from the project directory

## Development Workflow

The project follows a structured development approach:

1. **Models First**: Define entity models and relationships
2. **Database Setup**: Configure DbContext and connection
3. **Repository Layer**: Implement data access patterns
4. **Controllers**: Build MVC controllers with dependency injection
5. **Views**: Create responsive UI with Bootstrap
6. **Testing**: Write unit tests for all components
7. **Seeding**: Add sample data for demonstration

## Educational Value

This project is specifically designed for learning:

- **Real-world scenario** that students can relate to
- **Progressive complexity** - starts simple, adds features
- **Best practices** demonstrated throughout
- **Testing mindset** built in from the beginning
- **Modern web development** patterns and tools
