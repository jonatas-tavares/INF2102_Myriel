# Myriel - Property Lease Management System

**Myriel** is a web platform for managing commercial property leases, tailored to comply with French regulations. It is developed using **Django**, **PostgreSQL**, and **Docker**, and is designed for private deployment in shopping center environments.

![License](https://img.shields.io/badge/license-proprietary-red)



## Features

- Contract lifecycle management with periodic and prorated billing
- Commercial taxes (*Taxe de Bureaux*, *Taxe Foncière*) configuration
- Support for co-tenants, stores, and property inventories registration
- INSEE-based readjustments and customizable durations
- Role-based access: Owners, Tenants, Administrators, Operators...
- Automatic job scheduling for billing generation and contract readjustments

## Stack

- Python 3.10+
- Django 5.x
- PostgreSQL 16+
- Docker & Docker Compose
- Gunicorn + Nginx (production)
- APScheduler (for recurring tasks)

## Development

1. **Clone the repository**

   ```bash
   git clone git@github.com:TreeTechDev/myriel.git
   cd myriel
   ```

2. **Copy and configure environment variables**

   ```bash
   cp .env.example .env
   # Edit .env with local credentials
   # If necessary, ask for involved developers
   ```

3. **Build and run development containers**

   ```bash
   cd Locaux_project
   docker compose -f docker-compose.dev.yml up --build
   ```

4. **Access the application locally**
    While building, credentials for admin user will be shown in terminal, 
    be sure to start container build without `-d` option to get access
    in the system when first running.

   - Web: [http://localhost:8000](http://localhost:8000)
   - Admin: [http://localhost:8000/admin/](http://localhost:8000/admin/)

    If necessary, admin password can be changed manually using Django shell.

    The Docker stack now includes a dedicated ``scheduler`` service that runs
    ``python manage.py runapscheduler``. It uses a blocking scheduler and a
    persistent job store, so scheduled tasks continue running independently of
    the web server. Restart the ``scheduler`` container whenever code that
    defines jobs changes. The container waits for the ``web`` service health
    check before booting; you can customise the target host, port, and polling
    cadence through the ``SCHEDULER_WEB_*`` variables defined in `.env.example`.

## Running Tests

```bash
# Server must be running
cd Locaux_project
docker compose -f docker-compose.dev.yml exec web python manage.py test
```
**Running individual tests**: Scheduler job tests, located at `project/tests/jobs` can be run individually by using:

```bash
python manage.py test project.tests.jobs
```
## Bulk Data Import

The management command `bulk_insert` (see `project/management/commands/bulk_insert.py`) imports store or tenant records from CSV or XLSX spreadsheets. Execute it inside the **running** web container so it uses the same environment as the application:

```bash
cd Locaux_project
docker compose -f <docker-compose.dev.yml OR  docker-compose.prod.yml> exec web python manage.py bulk_insert --model store --file /code/imports/stores.xlsx
docker compose -f <docker-compose.dev.yml OR  docker-compose.prod.yml> exec web python manage.py bulk_insert --model tenant --file /code/imports/tenants.csv
```

Use the `--dry-run` flag to validate the dataset without writing to the database; the command reports any skipped rows and, for tenants, prints the temporary passwords generated at the end.

**Store imports** expect at least the columns `lot_code` and `property_name`. Additional fields such as `surface_area`, `store_type`, `state`, `desired_rent`, `floor`, or `stairs` are optional. The loader normalises headers so aliases like `lot`, `code_du_lot`, `surface`, `etat`, or `loyer` are accepted.

**Tenant imports** require `username`, `email`, `first_name`, `last_name`, `role`, `address`, `zip_code`, and `city`. Optional fields include `display_name`, `accounting_number`, `company_name`, `cell_phone`, `country`, and `status_account`. Role names must match existing Django groups (for example `LOCATAIRE`, `PROPRIETAIRE D'IMMEUBLES`, `GESTIONNAIRE D'IMMEUBLES`). When the spreadsheet contains multiple roles, separate them with commas or semicolons.

Ensure the spreadsheet is accessible from inside the container (for example by mounting an `imports/` directory) before running the command.

## Project Structure

```text
Locaux_project/
│
├── locaux/          # Django settings, auth, context processors
├── project/         # Core logic: models, views, forms, tests
├── scheduler/       # Scheduled tasks (APScheduler jobs)
├── logger/          # Logging and audit tracking persisted to database
├── templates/       # HTML templates organized by module
├── static/          # JS, CSS, and assets
├── docker/          # Configuration and definition files for Docker environments (prod/dev)
├── manage.py        # Django entry point
├── requirements.txt # Python dependencies
└── (...)
```
## *Deployment* (Production)

1. **Configure `.env.prod`** with production settings. The scheduler service
   reads the same environment file as the web container.

2. **Build and start the production stack:**

   ```bash
   docker compose -f docker-compose.prod.yml up -d --build
   ```

3. **Ensure Nginx is exposed via your domain:**

   The application should be accessible at:

   ```
   https://myriel.biobd.inf.puc-rio.br
   ```

4. **Use a valid SSL certificate (Let's Encrypt or custom)** and adjust `nginx.conf` accordingly.

## Licensing & Confidentiality

This project is **private and proprietary**.  
Unauthorized use, duplication, or distribution of any part of this software is strictly prohibited.

For licensing or commercial inquiries, contact the project maintainers.

## Development guidelines

- Use a clean `feature-branch` → `develop` workflow
- Follow PEP8 conventions (to be enforced via `black`, `flake8`, and `isort`)
- **Dont commit** `.env` or secret credentials
- Write meaningful commit messages

