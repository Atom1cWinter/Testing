# ScheduleBuddy – Class Project

ScheduleBuddy is a course recommendation system designed for UNCC students.  
The application recommends optimal future courses based on:

- A student's **major**
- **Completed coursework**
- **Degree progress**
- Catalog data imported from **CSV**

The project is written in **C# (.NET 8)** and follows SOLID design principles, with backend logic separated into the `ScheduleBuddy.Core` project and a front-end console UI in `ScheduleBuddy`.

SQLite is used to store:

- Courses  
- Majors  
- Users  
- Students  
- Course history  

The database is populated using `courses.csv` through a built-in **seeding command**.

---

## 🎯 Project Overview

**ScheduleBuddy** is a student-built tool that:

- Imports course data from `courses.csv` into a local SQLite database  
- Tracks students, majors, and completed courses  
- Provides a foundation for a recommendation engine that suggests future courses  
- Separates UI and core logic so the front end can be swapped (console now, GUI/web later)  

Authentication is handled by simple username + password (stored in plain text for this class project only) using the `AccountRepository` and `Authenticator` services.

---

## 🛠 Tech Stack

- **C# / .NET 8** – core language and runtime  
- **SQLite** – lightweight embedded database  
- **Microsoft.Data.Sqlite** – data access library  
- **JetBrains Rider** – primary IDE (works with Visual Studio as well)

---

## 📂 Project Structure

```text
3112-ScheduleBuddy/
│
├── identifier.sqlite        # SQLite database (required)
├── courses.csv              # CSV used for course seeding
├── ScheduleBuddy.sln
├── README.md
│
├── ScheduleBuddy/           # Console UI (entry point)
│   └── Program.cs
│
└── ScheduleBuddy.Core/      # Backend services, models, interfaces
```

> The database file **must remain in the repo root**.

---

## 🚀 Getting Started

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/Atom1cWinter/3112-ScheduleBuddy.git
cd 3112-ScheduleBuddy
```

Confirm these files exist in the repository root:

- `identifier.sqlite`
- `courses.csv`
- `ScheduleBuddy.sln`

If `identifier.sqlite` is missing, see **🔄 Recovering the Database**.

---

### 2️⃣ Open the Solution

You can use **JetBrains Rider** or **Visual Studio**.

**Rider:**

1. Open Rider  
2. **File → Open…** and select `ScheduleBuddy.sln`  
3. Let Rider restore NuGet packages  

---

## 🗄 Using SQLite in Rider (Optional but Recommended)

This step is only for browsing/editing the DB from the IDE—the application works without it as long as `identifier.sqlite` exists.

1. **View → Tool Windows → Database**  
2. Click **`+` → Data Source from Path…`**  
3. Select `identifier.sqlite` in the repo root  
4. Click **Test Connection**, then **OK**

You should now see tables like:

- `Courses`  
- `Majors`  
- `Users`  
- `Students`  
- `CourseHistory`  

---

## 🧱 Database Schema

If you ever need to recreate the DB file from scratch, use this schema.

Save this as `schema.sql` or paste directly into a Rider SQL console connected to `identifier.sqlite`.

```sql
-- Courses table
CREATE TABLE IF NOT EXISTS Courses (
    CourseId          INTEGER PRIMARY KEY AUTOINCREMENT,
    SubjectCode       TEXT    NOT NULL,
    CourseNumber      INTEGER NOT NULL,
    Code              TEXT    NOT NULL UNIQUE,
    Name              TEXT    NOT NULL,
    CreditsText       TEXT,
    CreditsMin        INTEGER,
    CreditsMax        INTEGER,
    PrerequisitesText TEXT,
    CorequisitesText  TEXT,
    CrosslistedText   TEXT,
    Description       TEXT
);

-- Majors table
CREATE TABLE IF NOT EXISTS Majors (
    MajorId         INTEGER PRIMARY KEY,
    MajorCode       TEXT,
    Name            TEXT    NOT NULL,
    TotalCreditsReq INTEGER
);

-- Users table (plain-text passwords for class project only)
CREATE TABLE IF NOT EXISTS Users (
    UserId      INTEGER PRIMARY KEY,
    Username    TEXT NOT NULL UNIQUE,
    Email       TEXT,
    PhoneNumber TEXT,
    Password    TEXT NOT NULL
);

-- Students table
CREATE TABLE IF NOT EXISTS Students (
    StudentId INTEGER PRIMARY KEY AUTOINCREMENT,
    UserId    INTEGER NOT NULL,
    MajorId   INTEGER,
    FOREIGN KEY (UserId) REFERENCES Users(UserId),
    FOREIGN KEY (MajorId) REFERENCES Majors(MajorId)
);

-- Course history for students
CREATE TABLE IF NOT EXISTS CourseHistory (
    CourseHistoryId INTEGER PRIMARY KEY AUTOINCREMENT,
    StudentId       INTEGER NOT NULL,
    CourseId        INTEGER NOT NULL,
    Grade           TEXT,
    Term            TEXT,
    FOREIGN KEY (StudentId) REFERENCES Students(StudentId),
    FOREIGN KEY (CourseId) REFERENCES Courses(CourseId)
);
```

**To run this in Rider:**

1. In the Database tool window, right-click `identifier.sqlite` → **New → Query Console**  
2. Paste the SQL above  
3. Click **Run (▶)**  

---

## 🌱 Seeding the Database (Loading Courses)

The console app supports a `seed` mode that reads `courses.csv` and populates the `Courses` table.

### Using the `dotnet` CLI

From the repo root:

```bash
cd ScheduleBuddy
dotnet run seed
```

Expected output:

```text
Reading CSV...
Loaded XXXX courses.
Seeding database...
Done seeding
```

### Using Rider

1. **Run → Edit Configurations…**  
2. Add a new **.NET Project** configuration targeting the `ScheduleBuddy` project  
3. Set **Program arguments** to:

   ```text
   seed
   ```

4. Run this configuration to execute the seeding process.

---

## 🔐 Authentication

Authentication is intentionally simple for this class project:

- User credentials are stored in the `Users` table as **plain-text passwords**  
- `AccountRepository` handles all database access for users:
  - Finding a user by ID or username
  - Creating new users
  - Updating passwords
- `Authenticator` coordinates login and registration:
  - `Authenticate(username, password)` returns a `User` or `null`
  - `Register(username, email, phoneNumber, password)` creates a new user (or returns existing)
  - `ChangePassword(userId, currentPassword, newPassword)` updates the password if the current one matches

> ⚠️ **Important:** Plain-text passwords are used *only* because this is a small student project.  
> In any real application you would use salted password hashes instead.

---

## ❗ Troubleshooting

### `no such table: Courses`

This usually means the app is connecting to a **different SQLite file** than the one you’re inspecting.

Check:

1. `identifier.sqlite` exists in the **repo root**  
2. Rider’s data source points to that same file  
3. In `Program.cs`, the DB path matches your folder layout, for example:

   ```csharp
   // Example: console project is in /ScheduleBuddy and DB is one level up
   string dbPath = "../identifier.sqlite";
   string connectionString = $"Data Source={dbPath}";
   ```

If another `identifier.sqlite` was created in a different folder, delete the stray file and rerun seeding.

---

### MalformedLineException while parsing `courses.csv`

This indicates a formatting problem in the CSV (unbalanced quotes, extra commas, etc.).

- The error message will include a **line number**  
- Open `courses.csv` in a text editor  
- Fix that line so it matches the structure of surrounding lines  

---

## 🔄 Recovering the Database

If `identifier.sqlite` is missing or corrupted:

### Option A – Get it from a teammate (recommended)

Ask another team member to send their copy of `identifier.sqlite` and place it in the repo root.

### Option B – Rebuild manually

1. Create an empty file named `identifier.sqlite` in the repo root  
2. Connect to it in Rider’s Database window  
3. Run the **schema.sql** script (see above)  
4. Seed courses:

   ```bash
   cd ScheduleBuddy
   dotnet run seed
   ```

---

## 🤝 Contributing

- Keep UI code in **`ScheduleBuddy`** and business logic in **`ScheduleBuddy.Core`**  
- Follow **SOLID** principles for new services and models  
- If you modify the schema or add tables:
  - Update **this README** (schema section)
  - Update any seeding logic and models  
- If you change CSV formats, update `DataReader` and verify seeding still works

---

## 📄 License

This project is intended for UNCC coursework and internal team collaboration.  
Redistribution outside the project group should be approved by the team.
