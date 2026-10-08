# 🧊 Smart Fridge & Nutrition Coach

Smart Fridge is a web application that helps users manage the food stored in their refrigerator. It lets them track expiration dates, identify products that should be used soon, and find nutritional information or recipe ideas based on the products they have available.

## Main features

- User registration, login, and profile management.
- Add, edit, and remove products from the fridge.
- Alerts for expired products and products that are close to expiring.
- Search for nutritional information and recipes.
- Nutrition tracking and meal-planning tools.
- Admin area for viewing statistics and managing users.
- Responsive interface with dark mode support.

## Prerequisites

- **Python 3.13**, as specified in the project's `.python-version` file.
- **Git** to clone the repository.
- **pip**, included with Python.
- A web browser.
- An Internet connection for external services (USDA and TheMealDB).

A USDA API key is required to search FoodData Central for nutritional information. The application can run without this key, but USDA searches will return no results. Recipes are provided by TheMealDB.

## Get and run the application

In a terminal, clone the repository and change to its directory:

```bash
git clone https://github.com/theoluthin2903/Theo_L_Maxime_Projet_Smart_Fridge.git
cd Theo_L_Maxime_Projet_Smart_Fridge
```

Create and activate a virtual environment.

**Windows (PowerShell):**

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
python3.13 -m venv .venv
source .venv/bin/activate
```

Install the dependencies and start the server:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m uvicorn main:app --reload
```

Then open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser. Interactive API documentation is available at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs).

### Optional configuration

By default, the application uses a local SQLite database named `smartfridge.db`, created in the project directory. To enable USDA searches, create a `.env` file in the project root and add your key:

```dotenv
USDA_API_KEY=your_usda_api_key
```

#### Get a USDA API key

1. Open the official [USDA FoodData Central API guide](https://fdc.nal.usda.gov/api-guide.html).
2. Follow the **Sign up to obtain a key** link to the data.gov registration form.
3. Complete the form with your email address and submit it. Your API key will be sent to that email address.
4. Copy the key into the `USDA_API_KEY` entry in your root `.env` file, as shown above, then restart the application.

Keep the key private: do not commit `.env` to Git or publish the key. The USDA documents a default rate limit of 1,000 requests per hour per IP address; exceeding it can temporarily block the key.

To use a PostgreSQL database hosted on Supabase instead of SQLite, also set `DATABASE_URL` in this file to the connection URL provided by Supabase:

```dotenv
DATABASE_URL=replace_with_the_postgresql_url_provided_by_supabase
```

The PostgreSQL driver is included in `requirements.txt`. Do not share your `.env` file: it may contain credentials or private keys.

### Create an administrator account

This step is optional. After installing the dependencies, run:

```bash
python create_admin.py
```

Enter an email address and a password of at least six characters when prompted. The password is not displayed while you type. The account can access the admin area at `/admin`.

### Use Supabase and migrate SQLite data

This is only necessary if you want to use Supabase. First, configure `DATABASE_URL` in `.env`. To test the connection:

```bash
python test_supabase_connection.py
```

To copy data from the local `smartfridge.db` database to the configured database:

```bash
python migrate_sqlite_to_supabase.py
```

The migration script requires an existing SQLite database and a valid Supabase URL.

## Known limitations

- Users must enter and keep their fridge contents up to date; the application does not detect food using sensors.
- Nutritional information and recipes depend on third-party services. Their availability, coverage, and results are not guaranteed by the application.
- Without `USDA_API_KEY`, USDA nutritional searches return no results. An Internet connection is also required to access external services.
- With the default SQLite setup, data is stored in the local `smartfridge.db` file. It is not shared between installations and must be backed up separately.
- This is an educational project. Before production use, the CORS configuration should be restricted and the application configuration, secrets, and database should be secured.

## Key files

```text
main.py                         FastAPI entry point, database table creation, and router registration
app/
  db/database.py                SQLite / PostgreSQL configuration and database access
  db/models.py                  SQLAlchemy data models
  core/                         Shared security, authentication, and business logic
  routers/                      API routes (authentication and profile)
  web/
    pages/                      Web page routes (fridge, recipes, alerts, admin, etc.)
    data.py                     Data access and external API calls
    nutrition_engine.py         Nutritional calculations
static/styles.css               Interface stylesheet
requirements.txt                Python dependencies
create_admin.py                 Create or promote an administrator account
migrate_sqlite_to_supabase.py   Migrate SQLite data to PostgreSQL / Supabase
test_supabase_connection.py     Test the Supabase database connection
Procfile                        Deployment startup command
```

## Deployed application

[Open Smart Fridge on Scalingo](https://theol-smartfridgeapp.osc-fr1.scalingo.io)

## Project brief

The full project brief is available in [`Project Brief Smart Fridge & Nutrition Coach (1).pdf`](./Project%20Brief%20Smart%20Fridge%20%26%20Nutrition%20Coach%20(1).pdf).

## Authors

- Théo LUTHIN
- Maxime HEINZ
