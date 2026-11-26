# Library Management System

A desktop application that helps a small library replace paper records and spreadsheets. It manages books and members, tracks borrowing and overdue loans, and turns the loan history into useful reports.

Built with **Python**, **Tkinter** and **SQLite** as the 2nd-year Python course project (2026/2027).

---

## Table of Contents

1. [Features](#features)
2. [Development Status](#development-status)
3. [Tech Stack](#tech-stack)
4. [Getting Started](#getting-started)
5. [Default Accounts](#default-accounts)
6. [Project Structure](#project-structure)
7. [Database Design](#database-design)
8. [Sample Data](#sample-data)
9. [Configuration](#configuration)
10. [Security](#security)
11. [Team](#team)
12. [AI Tools Usage](#ai-tools-usage)

---

## Features

| # | Feature | What it does |
|---|---|---|
| 1 | **Book management** | Add, edit, delete and view books linked to an author and a category, with total and available copies |
| 2 | **Member management** | Register library members and maintain their contact details |
| 3 | **Borrowing and returns** | Issue a book to a member, record its return, and update available copies automatically |
| 4 | **Overdue tracking** | Detect late books, count the days overdue and calculate the fine |
| 5 | **Search and filtering** | Search books by title or author; filter by category and availability |
| 6 | **Reports and statistics** | Most borrowed books, overdue books, loans per month, books per category |
| 7 | **Dashboard** | Library status at a glance: totals, active loans, overdue items and recent activity |
| 8 | **Login and roles** | Secure login with two roles: administrator and librarian |

## Development Status

> Tick each item when it is finished and merged. Remove this section before the final submission.

| Area | Status |
|---|---|
| Database schema, sample data, ERD file | Done |
| Login, password hashing, role-based menu | Done |
| Dashboard, theme and reusable widgets | Done |
| Books table (view) | Done |
| Books: search, filters, add / edit / delete | In progress |
| Members: search, add / edit / delete | In progress |
| Loans and returns screen, fines | In progress |
| Reports: overdue, loans per month, books per category | In progress |
| Documentation, ERD image, demo video | In progress |

## Tech Stack

| Technology | Role in the project |
|---|---|
| **Python 3.11+** | The entire application logic, data processing and reports |
| **Tkinter / ttk** | Graphical user interface (custom navy and gold theme) |
| **sqlite3** | Relational database with six related tables |
| **hashlib / hmac** | Password hashing (PBKDF2-SHA256) and safe verification |
| **datetime** | Due dates, overdue detection and fine calculation |

No external packages are required. Everything comes with a standard Python installation.

## Getting Started

### Requirements

- Python **3.11 or newer** (download from [python.org](https://www.python.org/downloads/) and tick *Add Python to PATH* on Windows)
- Tkinter (included with Python on Windows and macOS; on Linux run `sudo apt install python3-tk`)

Check your installation:

```bash
python --version
python -m tkinter        # should open a small test window
```

### Run the application

```bash
git clone <repository-url>
cd library_system
python main.py
```

On the first run, if `library.db` does not exist, it is created automatically and filled with sample data.

> On Windows you can also use `py main.py`. On macOS and Linux use `python3 main.py`.

### Reset the database

To rebuild the database from scratch with fresh sample data:

```bash
python -m database.seed_data
```

**Warning:** this deletes the existing `library.db`.

## Default Accounts

| Role | Username | Password | Access |
|---|---|---|---|
| Administrator | `admin` | `admin123` | Everything, including Reports & Statistics |
| Librarian | `librarian` | `lib123` | Dashboard, Books, Members, Loans & Returns |

## Project Structure

The code is split into three layers. Screens never write SQL; they call functions from `services`, and only `services` talk to the database.

```
library_system/
├── main.py                  # entry point: creates the window, switches screens
├── config.py                # loan period, fine per day, app title
├── requirements.txt
├── library.db               # SQLite database with sample data
│
├── database/                # data layer
│   ├── schema.sql           #   table definitions
│   ├── db.py                #   connection helper
│   └── seed_data.py         #   sample data generator
│
├── services/                # business logic + SQL queries
│   ├── auth.py              #   password hashing and login
│   ├── books.py             #   book operations
│   ├── members.py           #   member operations
│   ├── loans.py             #   borrowing, returns, overdue, fines
│   ├── reports.py           #   report queries
│   └── dashboard.py         #   dashboard numbers
│
├── gui/                     # presentation layer
│   ├── theme.py             #   colors, fonts and ttk styles
│   ├── widgets.py           #   reusable components (Page, DataTable, StatCard...)
│   ├── login.py             #   login screen
│   ├── main_window.py       #   sidebar navigation and role-based menu
│   ├── dashboard_frame.py
│   ├── books_frame.py
│   ├── members_frame.py
│   ├── loans_frame.py
│   └── reports_frame.py
│
└── docs/
    ├── erd.dbml             # ERD source (paste into dbdiagram.io)
    ├── AI_usage.md          # AI tools usage log
    └── Project_Documentation_Template.md
```

## Database Design

Six tables, all logically related. Dates are stored as ISO text (`YYYY-MM-DD`).

| Table | Purpose | Key columns |
|---|---|---|
| `users` | System accounts | `id` (PK), `username` (unique), `password_hash`, `role` (admin / librarian) |
| `members` | Library members | `id` (PK), `name`, `phone`, `email` (unique), `join_date` |
| `categories` | Book categories | `id` (PK), `name` (unique) |
| `authors` | Book authors | `id` (PK), `name` (unique) |
| `books` | The catalogue | `id` (PK), `title`, `author_id` (FK), `category_id` (FK), `publish_year`, `copies_total`, `copies_available` |
| `loans` | Borrowing records | `id` (PK), `book_id` (FK), `member_id` (FK), `issued_by` (FK), `loan_date`, `due_date`, `return_date` |

### Relationships

| Relationship | Type | Foreign key |
|---|---|---|
| One author has many books | 1 : N | `books.author_id` → `authors.id` |
| One category has many books | 1 : N | `books.category_id` → `categories.id` |
| One book has many loans | 1 : N | `loans.book_id` → `books.id` |
| One member has many loans | 1 : N | `loans.member_id` → `members.id` |
| One user issues many loans | 1 : N | `loans.issued_by` → `users.id` |

A loan with an empty `return_date` is still open. If it is also past its `due_date`, it is overdue.

Integrity rules: foreign keys are enforced, `role` is limited to `admin` / `librarian`, and `copies_available` must stay between 0 and `copies_total`.

The ERD source is in [`docs/erd.dbml`](docs/erd.dbml). Paste it into [dbdiagram.io](https://dbdiagram.io) to view or export the diagram.

## Sample Data

The delivered database lets every feature, search and report be tested immediately:

- **50** books across 7 categories (fiction, science, technology, history, self-development, philosophy, children)
- **20** members
- **100** loans: 85 returned, 7 currently borrowed and 8 overdue
- **2** user accounts (see [Default Accounts](#default-accounts))

Loan dates are generated relative to the day the database is created, so the overdue examples are always current.

## Configuration

Business settings live in [`config.py`](config.py):

| Setting | Default | Meaning |
|---|---|---|
| `LOAN_DAYS` | `14` | Days a book can be borrowed |
| `FINE_PER_DAY` | `2.0` | Fine (EGP) for each overdue day |
| `APP_TITLE` | `Library Management System` | Window title |

## Security

- Passwords are **never stored in plain text**. Each password is hashed with PBKDF2-SHA256 (100,000 iterations) and a random per-user salt, then compared with a constant-time function.
- All SQL queries use **parameter placeholders** (`?`), which prevents SQL injection.
- Foreign keys are enabled on every connection.
- Admin-only screens are hidden from the librarian role.

## Team

| Name | Edu ID | Section | Responsibility |
|---|---|---|---|
| _Member 1_ | | | Database design, sample data, ERD, theme and widgets |
| _Member 2_ | | | Authentication, roles, password security, main window |
| _Member 3_ | | | Books module: CRUD, search and filtering |
| _Member 4_ | | | Members, borrowing and returns, overdue and fines |
| _Member 5_ | | | Reports, dashboard, documentation, demo video, integration |

## AI Tools Usage

AI tools were used as learning and coding assistants. Every team member understands and can explain the code they submitted. The tools used, the components they helped with and the type of assistance are documented in [`docs/AI_usage.md`](docs/AI_usage.md).

---

_Python Course Project, 2nd year, 2026/2027._
