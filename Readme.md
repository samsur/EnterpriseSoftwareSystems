# 🎓 PROG30000 - Enterprise Software Systems

Welcome to the exercises repository for **PROG30000 - Enterprise Software Systems** course! This repository contains hands-on projects and exercises designed to help you master enterprise-level software development concepts.

## 📚 Table of Contents

- [Working With Data](#-working-with-data)
- [Project Setup](#-project-setup)
- [Unit Testing](#-unit-testing)
- [Prerequisites](#-prerequisites)

---

## 🗄️ Working With Data

This is an **ASP.NET Core MVC** project that demonstrates how to use **Entity Framework Core** to perform CRUD operations (Create, Read, Update, Delete) with a **SQLite database**.

### ✨ Features
- 📖 **Read** data from SQLite database
- ✏️ **Update** existing records
- 💾 **Save** new data entries
- 🗑️ **Delete** data records
- 🧪 **Unit testing** with xUnit and Moq

---

## 🚀 Project Setup

### 📝 Prerequisites
- .NET 9.0 SDK
- Visual Studio 2022 or VS Code
- Basic knowledge of C# and ASP.NET Core

### 🛠️ Create Your Solution and Web Project

1. **Open your command prompt, PowerShell, or terminal**
2. **Navigate to your desired project location**
3. **Run the following commands one at a time:**

```bash
# Create global.json with SDK version 9.0
dotnet new globaljson --sdk-version 9.0 --output DataSln/WorkingWithData

# Create ASP.NET Core MVC project (no HTTPS for local development)
dotnet new mvc --no-https --output DataSln/WorkingWithData --framework net9.0

# Create solution file
dotnet new sln -o DataSln

# Add the WorkingWithData project to the solution
dotnet sln DataSln add DataSln/WorkingWithData
```

### 📋 What These Commands Do:

| Command | Purpose |
|---------|---------|
| `dotnet new globaljson` | Creates a `global.json` file specifying .NET SDK version 9.0 |
| `dotnet new mvc` | Creates an ASP.NET Core MVC project in the specified folder |
| `dotnet new sln` | Creates a solution file (`DataSln.sln`) |
| `dotnet sln add` | Adds the WorkingWithData project to the solution |

---

## 🧪 Unit Testing

### 🔧 Create a Unit Test Project

Run the following commands to set up unit testing:

```bash
# Create xUnit test project
dotnet new xunit -o DataSln/WorkingWithData.Tests --framework net9.0

# Add test project to solution
dotnet sln DataSln add DataSln/WorkingWithData.Tests

# Add reference from test project to main project
dotnet add DataSln/WorkingWithData.Tests reference DataSln/WorkingWithData

# Install Moq framework for mocking
dotnet add DataSln/WorkingWithData.Tests package Moq --version 4.18.4
```

### 📦 Testing Framework
The unit test project uses:
- **xUnit** - Testing framework
- **Moq** - Mocking framework for creating test doubles

---

## 📋 Prerequisites

- **.NET 9.0 SDK** - [Download here](https://dotnet.microsoft.com/download)
- **SQLite** - Lightweight database engine
- **Entity Framework Core** - ORM for .NET
- **Visual Studio 2022** or **Visual Studio Code**

---

## 🎯 Learning Objectives

By completing these exercises, you will learn:
- ✅ Setting up ASP.NET Core MVC projects
- ✅ Implementing Entity Framework Core with SQLite
- ✅ Creating and managing database connections
- ✅ Performing CRUD operations
- ✅ Writing unit tests with xUnit and Moq
- ✅ Following enterprise software development best practices

---

*Happy coding! 🚀*