# Rally Supply POS · TFA4

**Technical Formative Assessment 4: Sessions and Authentication**

Rally Supply carries the TFA3 pickleball shop and POS into TFA4. Staff sign in
with a username and password; a CodeIgniter filter protects customer and user
account management. Password hashes, the MySQL database, and browser sessions
are included in this project’s setup.

## Cover sheet

- Student/Group name: **[Complete before submission]**
- Member 1 and role: **[Complete before submission]**
- Member 2 and role: **[Complete before submission]**
- Section: **[Complete before submission]**
- Professor: **[Complete before submission]**

## Submission links

- GitHub repository: **[Add after creating the repository]**
- Hosted application: **[Add after deployment]**

## Demo sign-in

- Username: `courtadmin`
- Password: `RallyDemo4!`

This shared password is for the local classroom demonstration only. The
`.env.example` contains the same demo default so the seeder can hash it for
each fictional staff account. Change it in `.env` before seeding a private
copy. The database and SQL export contain only password hashes.

## Features

- Customer and staff account rosters, search, sorting, create, edit, and delete.
- Unique staff usernames, active/disabled sign-in state, and hashed passwords.
- Login with `password_verify()`, session ID regeneration, a generic error, and
  a five-attempt session lockout.
- A `staff` filter on protected routes, no-store headers for staff responses,
  POST + CSRF logout, and a safe post-login destination allowlist.
- The TFA3 shop, cart, checkout, inventory, orders, POS sales, and avatar
  thumbnail workflow remain in the TA4 copy.
- MySQL migrations and an idempotent seeder, plus a phpMyAdmin-ready database
  export at `database/rally_supply.sql`.

## Technology

- PHP 8.2 or newer; tested here with PHP 8.2.12.
- CodeIgniter 4.7.4.
- MySQL or MariaDB; locally verified with XAMPP MariaDB 10.4.32.
- PHPUnit 10.5.65.
- Vanilla JavaScript and local CSS; no JavaScript framework or build step.

## Run with the included XAMPP launcher

1. Open this `pickleball-pos-ta4` folder in VS Code or File Explorer.
2. Double-click `Start-TA4.cmd` and leave the terminal open.
3. Visit [http://localhost:8085/](http://localhost:8085/).
4. Sign in at [http://localhost:8085/login](http://localhost:8085/login) with
   the demo credentials above.
5. Open the TA4 database in [phpMyAdmin](http://localhost/phpmyadmin/index.php?route=/database/structure&db=rally_supply_ta4&server=3).
   The selected server is **Rally Supply TA4 (3309)** and the database is
   `rally_supply_ta4`.
6. Press Ctrl+C in the launcher terminal to stop the site. Run
   `Stop-TA4-Database.cmd` to stop its database; saved records remain in
   `.local/mysql`.

The launcher uses a separate local MariaDB data folder and port so the TA3
project and the default XAMPP database are not overwritten. It creates the
private `.env`, runs migrations, and seeds sample accounts and products on each
start. Re-running the seeder keeps existing roster/catalog records and resets
seeded staff passwords to the `.env` demo password.

## Set up manually

### Option A: migrations and seeder

1. Install PHP 8.2+, Composer, MySQLi, `intl`, `mbstring`, and GD.
2. Create a new MySQL database named `rally_supply_ta4` with `utf8mb4`.
3. Copy `.env.example` to `.env`; set `app.baseURL`, database host, database
   name, username, password, and port for your environment.
4. Set `demo.password` in `.env` to a classroom-only password of at least eight
   characters.
5. Run `composer install`.
6. Run `php spark migrate --all`.
7. Run `php spark db:seed RallySupplySeeder`.
8. Run `php spark serve --host 127.0.0.1 --port 8085`.
9. Open `http://localhost:8085/` and sign in as `courtadmin` with the
   `demo.password` value.

### Option B: import the full SQL export

1. Create and select a new `rally_supply_ta4` database in phpMyAdmin.
2. Choose **Import**, select `database/rally_supply.sql`, and run the import.
3. Copy `.env.example` to `.env` and enter the same database connection details.
4. Run `composer install`, then start the site with `php spark serve --host
   127.0.0.1 --port 8085`.
5. Sign in with the demo credentials above. The SQL file contains the sample
   data and matching password hashes; it does not contain plaintext passwords.

## How authentication works

1. A staff member submits the login form over POST with the CSRF token.
2. The Auth controller looks up the username and checks the submitted password
   against `users.password` with `password_verify()`.
3. On success, CodeIgniter regenerates the session ID and stores only the staff
   ID and display name in the session.
4. The `staff` filter checks that the session ID belongs to an active user
   before the staff controller runs. A signed-out request goes to `/login`
   with a safe return path and a short explanation.
5. Logout is POST-only, clears the login data, destroys the session, and returns
   to `/login` with a signed-out notice.

The user Model hashes non-empty passwords before insert and update. An empty
password on the edit form means “keep the existing password.” The user list and
edit view never render the stored hash.

## Route access table

See [`docs/Feature-Map.md`](docs/Feature-Map.md) for the complete route table,
including every public and protected GET/POST route and its handler.

## Verify the project

The PHPUnit feature tests use a separate `rally_supply_ta4_test` schema, not
the demo database. Start the local database first, then run:

```text
composer test
```

See [`docs/Verification.md`](docs/Verification.md) for the manual test script
and the most recent real test output.

## Project structure

- `app/Controllers/`: account forms, authentication, shop, cart, and POS flow.
- `app/Filters/StaffOnly.php`: active-user check and private response headers.
- `app/Models/`: database access and password hashing callbacks.
- `app/Database/Migrations/`: schema, credential consolidation, and sessions.
- `app/Database/Seeds/RallySupplySeeder.php`: demo roster and product catalog.
- `app/Views/accounts/` and `app/Views/staff/`: account and sign-in pages.
- `public/assets/`: site styles, JavaScript, images, and prepared avatars.
- `database/rally_supply.sql`: complete importable MySQL export.
- `tests/Feature/`: login, session, and access-control feature tests.
- `docs/`: implementation map, verification notes, and deployment steps.

## Deployment and known limits

The app needs PHP 8.2+, MySQL/MariaDB, and a web server whose document root
points to `public/`. See [`docs/DEPLOY.md`](docs/DEPLOY.md) for the Docker
configuration and deployment checklist. Public hosting and submission URLs
are intentionally left for the later submission step.

This is a classroom demo. All sample accounts share one documented demo
password; the current five-attempt lockout is stored in the browser session and
can be reset by clearing that session. A production deployment should replace
the demo password, use a private database account, configure HTTPS and a
production encryption key, and review rate limiting and account recovery.

## CodeIgniter references

- [Sessions](https://codeigniter.com/user_guide/libraries/sessions.html)
- [Controller filters](https://codeigniter.com/user_guide/incoming/filters.html)
