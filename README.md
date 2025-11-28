ScheduleBuddy – Class Project
ScheduleBuddy is a course recommendation system designed for UNCC students.
The application recommends optimal future courses based on:


A student's major


Completed coursework


Degree progress


Catalog data imported from CSV


The project is written in C# (.NET 8) and follows SOLID design principles, with backend logic separated into the ScheduleBuddy.Core project and a front-end console UI in ScheduleBuddy.
SQLite is used to store:


Courses


Majors


Users & login credentials


Students


Course history


The database is populated using courses.csv through a built-in seeding command.

🎯 Project Overview
ScheduleBuddy is a student-built tool that:


Imports course data from a CSV file and stores it in a local SQLite database


Tracks students, majors, and completed courses


Provides a foundation for a recommendation engine that suggests future courses


Separates UI and core logic so the front end can be swapped (console now, GUI/web later)


Implements simple username/password authentication (plain-text passwords for class/demo purposes)



🛠 Tech Stack


C# / .NET 8 – core language and runtime


SQLite – lightweight embedded database


Microsoft.Data.Sqlite – ADO.NET provider for SQLite


JetBrains Rider – primary IDE (works with Visual Studio as well)



📂 Project Structure
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
    ├── Interfaces/
    ├── Models/
    └── Services/


The database file must remain in the repo root.


🔐 Authentication Overview
Authentication is intentionally simple for this class project:


User credentials are stored in the Users table with plain-text passwords (no hashing, by design for MVP).


AccountRepository (in ScheduleBuddy.Core.Services) is responsible for:


Looking up users by ID or username


Creating new users


Updating passwords




Authenticator (in ScheduleBuddy.Core.Services) is responsible for:


Validating username + password combinations


Handling registration (avoid duplicate usernames)


Handling password changes via AccountRepository




This separation keeps database access in the repository and business logic in the authenticator while staying easy to reason about for the project.

🚀 Getting Started
1️⃣ Clone the Repository
git clone https://github.com/Atom1cWinter/3112-ScheduleBuddy.git
cd 3112-ScheduleBuddy

Confirm these files exist in the repository root:


identifier.sqlite


courses.csv


ScheduleBuddy.sln


If identifier.sqlite is missing, see 🔄 Recovering the Database.

2️⃣ Open the Solution
You can use JetBrains Rider or Visual Studio.
Rider:


Open Rider


File → Open… and select ScheduleBuddy.sln


Let Rider restore NuGet packages



🗄 SQLite Database in Rider (Optional but Recommended)
This step is only for browsing/editing the DB from the IDE—the application works without it as long as identifier.sqlite exists.


In Rider: View → Tool Windows → Database


Click + → Data Source from Path…`


Select identifier.sqlite in the repo root


Click Test Connection, then OK


You should now see tables like:


Courses


Majors


Users


Students


CourseHistory



🧱 Database Schema
If you ever need to recreate the DB file from scratch, use this schema.
Save as schema.sql or paste directly into a Rider SQL console connected to identifier.sqlite.
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

-- Users table (plain-text passwords for class/demo purposes)
CREATE TABLE IF NOT EXISTS Users (
    UserId      INTEGER PRIMARY KEY AUTOINCREMENT,
    Username    TEXT    NOT NULL UNIQUE,
    Email       TEXT,
    PhoneNumber TEXT,
    Password    TEXT    NOT NULL
);

-- Students table (links Users to Majors)
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
    FOREIGN KEY (CourseId)  REFERENCES Courses(CourseId)
);

To run this in Rider:


In the Database tool window, right-click identifier.sqlite → New → Query Console


Paste the SQL above


Click Run (▶)



🌱 Seeding the Database (Loading Courses)
The console app supports a seed mode that reads courses.csv and populates the Courses table.
Using the dotnet CLI
From the repo root:
cd ScheduleBuddy
dotnet run seed

Expected output:
Reading CSV...
Loaded XXXX courses.
Seeding database...
Done seeding

Using Rider


Run → Edit Configurations…


Add a new .NET Project configuration targeting the ScheduleBuddy project


Set Program arguments to:
seed



Run this configuration to execute the seeding process.



🔐 Testing Authentication (example)
The console UI includes wiring for authentication using AccountRepository and Authenticator.
At a high level, the flow is:


User registers ⇒ Authenticator.Register(...)


Credentials are stored in Users table via AccountRepository.CreateUser(...)


Login ⇒ Authenticator.Authenticate(username, password)


Password changes ⇒ Authenticator.ChangePassword(...) which calls AccountRepository.UpdatePassword(...)


For the MVP, validation is intentionally light and passwords are stored in plain text.

❗ Troubleshooting
Error: no such table: Courses
This usually means the app is connecting to a different SQLite file than the one you’re inspecting.
Check:


identifier.sqlite exists in the repo root


Rider’s data source points to that same file


In Program.cs, the DB path matches your folder layout, for example:


// Example: console project is in /ScheduleBuddy and DB is one level up
string dbPath = "../identifier.sqlite";
string connectionString = $"Data Source={dbPath}";

If another identifier.sqlite was created in a different folder, delete the stray file and rerun seeding.

Error: MalformedLineException while parsing courses.csv
This indicates a formatting problem in the CSV (unbalanced quotes, extra commas, etc.).


The error message will include a line number


Open courses.csv in a text editor


Fix that line so it matches the structure of surrounding lines



🔄 Recovering the Database
If identifier.sqlite is missing or corrupted:
Option A – Get it from a teammate (recommended)
Ask another team member to send their copy of identifier.sqlite and place it in the repo root.
Option B – Rebuild manually


Create an empty file named identifier.sqlite in the repo root


Connect to it in Rider’s Database window


Run the schema.sql script (see above)


Seed courses:


cd ScheduleBuddy
dotnet run seed


🤝 Contributing


Keep UI code in the ScheduleBuddy project and business logic in ScheduleBuddy.Core


Follow SOLID principles for new services and models


If you modify the schema or add tables:


Update schema.sql


Update this README if setup changes




If you change CSV formats, update DataReader and verify seeding still works


When extending authentication, consider migrating from plain-text passwords to hashed+salted storage



📄 License
This project is intended for UNCC coursework and internal team collaboration.
Redistribution outside the project group should be approved by the team.
