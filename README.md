# Personal Finance & Expense Tracker

A full-stack web app to track income and expenses, set monthly budgets, and see where your money goes through interactive charts. It also shows how to use custom PostgreSQL aggregation queries behind a REST API.

Tech stack: React (Vite) · Recharts · Java 21 / Spring Boot 3 · Spring Data JPA · PostgreSQL 16 · Flyway · Docker · GitHub Actions · Kubernetes

<!-- Add screenshots here after you run the app: ![Dashboard](docs/dashboard.png) -->
Features
Categorized transactions: add, edit, delete and filter income and expenses by month, type and category
Recurring monthly budgets: set a spending limit per category that applies every month
Custom SQL reports: monthly category breakdown, budget vs actual, and 6-month income vs expense trend
Visual dashboard: pie chart, bar charts and budget progress meters (red when over budget)
Statement export: download any month as CSV or PDF
Safe money handling: amounts are stored as integer paise, never floating point
Architecture
React UI (:3000)  ->  Spring Boot REST API (:8080)  ->  PostgreSQL
   Recharts            controllers + JPA + native SQL      Flyway migrations

In development, Vite proxies /api calls to the backend. In Docker and Kubernetes, nginx does the same.

Project structure
backend/     Spring Boot service (entities, repositories, controllers, Flyway SQL)
frontend/    React app (App.jsx, Charts.jsx, api.js)
k8s/         Kubernetes manifests
.github/     CI/CD workflow (test, build, push Docker images)
docker-compose.yml
Getting started

Option 1: Docker (one command)
Requires Docker Desktop.

bash
docker compose up --build

Open http://localhost:3000. The database is created and filled with 6 months of demo data automatically.

Option 2: Run each part locally

Requires Java 21 (or 17, by changing java.version in backend/pom.xml), Maven 3.9+, Node 20+ and a PostgreSQL database.

1. Database. Start PostgreSQL, then create the user and database:

sql
CREATE USER finance_user WITH PASSWORD 'secretpassword';
CREATE DATABASE finance_db OWNER finance_user;

2. Backend (PowerShell example; change the port to match your database):

powershell
cd backend
$env:DB_URL="jdbc:postgresql://localhost:5432/finance_db"
$env:DB_PASSWORD="secretpassword"
mvn spring-boot:run

Flyway creates the tables and demo data on first start. The API runs on http://localhost:8080.

3. Frontend (in a second terminal):
   
bash
cd frontend
npm install
npm run dev

Open http://localhost:3000.

Configuration
Variable	Default	Purpose
DB_URL	jdbc:postgresql://localhost:5432/finance_db	Database connection string
DB_USER	finance_user	Database user
DB_PASSWORD	secretpassword	Database password. Change this before deploying


REST API
All amounts are in paise (100 paise = ₹1). Months look like 2026-11.

Method	Endpoint	Purpose
GET, POST	/api/transactions?month=&type=&category=	List (filtered) or add a transaction
PUT, DELETE	/api/transactions/{id}	Edit or delete a transaction
GET, PUT	/api/budgets	List budgets or set a category's monthly limit
GET	/api/reports/category-breakdown?month=	Spending per category with % share
GET	/api/reports/budget-vs-actual?month=	Budget limit vs amount spent
GET	/api/reports/monthly-trend?month=&months=6	Income vs expenses per month
GET	/api/export/csv?month=	Download CSV statement
GET	/api/export/pdf?month=	Download PDF statement
SQL highlights

The reports are native PostgreSQL queries in TransactionRepository.java:

Category breakdown: GROUP BY with a window function (SUM(SUM(...)) OVER ()) to get each category's percentage of total spending
Budget vs actual: a LEFT JOIN from budgets to transactions, so categories with a budget but no spending still appear
Monthly trend: conditional aggregation with SUM(...) FILTER (WHERE ...) and date_trunc('month', ...)


CI/CD and deployment
GitHub Actions (.github/workflows/ci.yml) runs backend tests, builds the frontend, and on main builds and pushes Docker images to GitHub Container Registry.
Kubernetes manifests are in k8s/app.yaml. Replace the image names and create the db-secrets secret first (instructions are at the top of that file).


Troubleshooting
invalid value for parameter "TimeZone": "Asia/Calcutta" on Windows: the backend's Java reports an old timezone name that PostgreSQL 16 rejects. Fix it with:
powershell
$env:JAVA_TOOL_OPTIONS="-Duser.timezone=UTC"

Port already in use: change the left-hand number in the ports: entry in docker-compose.yml, for example "5434:5432".
Frontend shows "Cannot reach the server": make sure the backend is running on port 8080.

Roadmap
User login (email or phone with JWT)
Recurring transactions (such as rent)
Automated tests
Budget alerts by email
