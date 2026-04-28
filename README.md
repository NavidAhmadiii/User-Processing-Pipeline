اینجا یه `README.md` کامل و انگلیسی که می‌تونی بدون تغییر کپی‌پیست کنی. تصویر `image.png` که توی پوشه‌ته رو هم همین‌جوری لینک دادم (اسم فایل رو دقیقاً `image.png` بذار کنار README).

```markdown
# User Processing Airflow DAG

This project is my first hands-on Airflow pipeline. It fetches a fake user profile from a live JSON API, transforms the data, writes it to a CSV file, and bulk-loads it into a PostgreSQL database using the `COPY` command. The DAG also includes a sensor that waits for the API to be available before proceeding.

## DAG Graph

![User Processing DAG](image.png)

## How It Works

1. **create_table** – Creates the `users` table in PostgreSQL if it doesn't already exist.
2. **is_api_available** – A sensor that pokes an external JSON API every 30 seconds (up to 5 minutes) until it returns a 200 status. The fetched data is passed downstream via XCom.
3. **extract_user** – Extracts only the needed fields (`id`, `firstname`, `lastname`, `email`) from the raw JSON.
4. **process_user** – Adds a `created_at` timestamp and saves the user record to a local CSV file (`/tmp/user_info.csv`).
5. **store_user** – Uses `PostgresHook.copy_expert` to bulk-insert the CSV into the `users` table in PostgreSQL.

## Prerequisites

- Apache Airflow 2.5+
- PostgreSQL database accessible from the Airflow environment
- Airflow providers:
  - `apache-airflow-providers-postgres`
  - `apache-airflow-providers-common-sql`
- Python library:
  - `requests`

Install dependencies with:
```bash
pip install -r requirements.txt
```

## Configuration

### Airflow Connection
Set up a PostgreSQL connection in Airflow with the ID `postgres`:
- **Conn Type**: Postgres
- **Host**: your database host
- **Schema**: your database name
- **Login**: your username
- **Password**: your password
- **Port**: 5432 (default)

### DAG Location
Place the `user_processing.py` file inside your Airflow `dags` folder, ideally inside a subfolder called `user-processing`. Airflow automatically discovers DAGs in all subfolders.

```
dags/
  user-processing/
    user_processing.py
```

## File Structure

```
user-processing-dag/
├── user_processing.py      # The DAG definition
├── requirements.txt        # Python dependencies
├── README.md               # This file
└── image.png               # Screenshot of the DAG graph
```

## Usage

1. Ensure the Airflow scheduler and webserver are running.
2. Place the repository contents inside the `dags` folder of your Airflow installation.
3. In the Airflow UI, locate the DAG named `user_processing`.
4. Trigger the DAG manually (or set a schedule when ready).
5. Monitor the run; the DAG will wait for the API to respond, then load data into PostgreSQL.

After a successful run, check the `users` table in your PostgreSQL database – it will contain the fetched fake user record.

## Notes

- The current DAG has no schedule and runs only on manual trigger. To automate, add a `schedule` parameter (e.g., `@daily`) to the `@dag` decorator.
- The CSV file `/tmp/user_info.csv` is written and read on the same worker. For multi-worker production setups, consider using an in-memory buffer or object storage.
- Sensitive data (passwords, connection strings) should be stored in Airflow Connections or a secrets backend, never hard-coded in DAG files.

## License

This project is for learning purposes. Feel free to use and modify.
```
