# 🧊 Smart Fridge & Nutrition Coach

A web application that lets you manage the contents of your refrigerator, track expiration dates, receive alerts, and get nutritional information and recipe ideas based on the products you have available.

Project created by **Théo L.** and **Maxime**.

## ✨ Features

- **User authentication** (registration, login, profile management).
- **Fridge management**: add, edit, and delete stored products.
- **Product catalog** with nutritional data retrieval (via the USDA API) and associated recipes (via TheMealDB).
- **Automatic alerts** for expired or soon-to-expire products.
- **Nutrition coach** based on the products currently in the fridge.
- **Admin panel** with:
  - statistics (users, admins, products, expired/soon-to-expire products);
  - chart of account creations over the last 7 days;
  - product breakdown by category;
  - activity log (`admin_logs`);
  - user management (role, deletion, password reset);
  - quick actions and automatic alerts.
- **Responsive** interface, **dark mode** compatible.

## 🛠️ Tech Stack

- **Backend**: [FastAPI](https://fastapi.tiangolo.com/) (Python)
- **ORM / Database**: **Supabase**
- **Frontend**: server-side rendered pages, styled with Tailwind CSS
- **Authentication**: middleware + JWT (Bearer) support documented in the OpenAPI schema
- **External APIs**: USDA FoodData Central (nutritional data), TheMealDB (recipes)

## 📁 Project Structure

```
.
├── app/                 # Application code (routers, web pages, database, business logic)
├── routers/             # API routes
├── static/              # Static files (CSS, JS, images)
├── main.py              # FastAPI application entry point
├── create_admin.py      # Script to create an administrator account
├── migrate_sqlite_to_supabase.py   # SQLite → Supabase migration script
├── test_supabase_connection.py     # Supabase connection test script
├── requirements-supabase.txt       # Supabase-specific dependencies (psycopg, python-dotenv)
└── Project Brief Smart Fridge & Nutrition Coach.pdf   # Project requirements document
```

## 🚀 Installation

### Prerequisites

- Python 3.10+
- pip

### Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/theoluthin2903/Theo_L_Maxime_Projet_Smart_Fridge.git
   cd Theo_L_Maxime_Projet_Smart_Fridge
   ```

2. Create and activate a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows: venv\Scripts\activate
   ```

3. Install the project dependencies (see `requirements.txt` at the root):
   ```bash
   pip install -r requirements.txt
   ```

4. (Optional) If you want to use **Supabase** as your database instead of SQLite, also install:
   ```bash
   pip install -r requirements-supabase.txt
   ```
   then configure your environment variables (e.g. `DATABASE_URL`) in a `.env` file at the project root.

5. Run the application:
   ```bash
   uvicorn main:app --reload
   ```
   ou
   ```bash
   python -m uvicorn main:app --reload
   ```
   The application is then available at [http://127.0.0.1:8000].

## 👑 Creating an Administrator Account

```bash
python create_admin.py
```

The password is not displayed while you type it in the terminal: just type it normally, then press Enter.

Then log in with this account and go to `/admin` to access the admin panel.

## ☁️ Migrating to Supabase

The project can run with SQLite (default) or with a PostgreSQL database hosted on Supabase.

1. Configure your Supabase connection in `.env`.
2. Test the connection:
   ```bash
   python test_supabase_connection.py
   ```
3. Migrate your existing data from SQLite:
   ```bash
   python migrate_sqlite_to_supabase.py
   ```

> ℹ️ The `admin_logs` table is created automatically via `Base.metadata.create_all(bind=engine)` when the application starts. If your project already has a migration system, remember to add this table to it.

## 📄 Project Documentation

The full project brief is available in the file [`Project Brief Smart Fridge & Nutrition Coach.pdf`](./Project%20Brief%20Smart%20Fridge%20%26%20Nutrition%20Coach%20(1).pdf).

## Link to the app deployed on Scalingo

https://theol-smartfridgeapp.osc-fr1.scalingo.io

## 👥 Authors

- Théo LUTHIN
- Maxime HEINZ
