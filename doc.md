# Laravel REST API — Setup Guide

Setting up the Laravel-based REST API from scratch. The API exposes a `users` resource and uses Laravel Sanctum for token-based authentication. Stack: **Laravel 11**, **Sanctum**, **MySQL 8**.

---

## Prerequisites

Before starting, ensure the following are installed on your machine:

- PHP >= 8.2
- Composer >= 2.x
- MySQL >= 8.0
- A database client (TablePlus, DBeaver, or MySQL Workbench)
- (Optional) Postman or Insomnia for testing endpoints

---

## 1. Project Setup

Create the Laravel project and open it in your editor:

```bash
composer create-project laravel/laravel api
cd api
code .
```

Install the API scaffold. This publishes the Sanctum configuration and adds the `routes/api.php` file:

```bash
php artisan install:api
```

After running this command, Laravel registers the `sanctum` middleware and creates `routes/api.php`.

![Sanctum installed — output confirming the API scaffold was published](./assets/image.png)
![routes/api.php created with the default Sanctum token route](./assets/image-1.png)

---

## 2. Database Connection

Open `.env` at the root of the project. Locate the `DB_*` block (lines 24–28) and fill in your local database credentials:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_database_name
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

![.env file with the DB block filled in with local credentials](./assets/image-4.png)

### Create the database

Before running migrations, create an empty database in MySQL matching the name you set in `DB_DATABASE`. Use your database client or run:

```sql
CREATE DATABASE your_database_name;
```

![Empty database created in the client — no tables yet](./assets/image-5.png)
![MySQL client showing the new database selected](./assets/image-6.png)
![Schema view confirming the database is empty before migration](./assets/image-7.png)

---

## 3. Migrations

Laravel ships with default migrations for `users`, `password_reset_tokens`, `sessions`, and `cache`. Run them to create the schema:

```bash
# State before migration — no tables exist yet
```

![Database client showing no tables before running migrate](./assets/image-9.png)

```bash
php artisan migrate
```

![Terminal output confirming all migrations ran successfully](./assets/image-10.png)
![Database client showing the default Laravel tables now created](./assets/image-11.png)

---

## 4. Seeders — Populate the Users Table

The `DatabaseSeeder` uses a factory to generate fake users. Open `database/seeders/DatabaseSeeder.php` and make the following changes:

1. Remove the example code block (highlighted in the screenshot below).
2. Uncomment the `User::factory()` call and set the number of records to generate.

```php
// Example: generate 10 fake users
User::factory(10)->create();
```

![DatabaseSeeder before edit — example code to remove is highlighted](./assets/image-2.png)
![DatabaseSeeder after edit — User factory call active with quantity set](./assets/image-3.png)

Run the seeder:

```bash
# Users table before seeding — empty
```

![Users table empty before running the seeder](./assets/image-12.png)

```bash
php artisan db:seed
```

![Terminal output showing the seeder executing and generating records](./assets/image-13.png)
![Users table after seeding — 10 fake users inserted](./assets/image-14.png)

---

## 5. Resource Controller

Generate a resource controller for the `User` model. The `-r` flag scaffolds all seven RESTful methods (`index`, `show`, `store`, `update`, `destroy`, etc.):

```bash
php artisan make:controller Api/UserController -r
```

Open `app/Http/Controllers/Api/UserController.php` and inject the `User` model via route-model binding in `show`, `update`, and `destroy`.

![Generated UserController with default empty method stubs](./assets/image-21.png)
![UserController after injecting the User model](./assets/image-22.png)

---

## 6. API Routes

Open `routes/api.php`. By default it only contains the Sanctum token route. Register the users resource:

![routes/api.php default state — only the Sanctum token route present](./assets/image-18.png)

Add the following import and route registration:

```php
use App\Http\Controllers\Api\UserController;

Route::apiResource('users', UserController::class);
```

![routes/api.php after adding the apiResource route for UserController](./assets/image-19.png)

Each controller method maps to a standard HTTP endpoint. Below are the implementations:

**index** — returns a paginated list of users

![UserController@index returning all users as JSON](./assets/image-20.png)

**store** — creates a new user

![UserController@store validating and persisting a new user](./assets/image-90.png)

**show** — returns a single user by ID

![UserController@show returning a single user via route-model binding](./assets/image91.png)

**update** — updates an existing user

![UserController@update applying validated changes to a user](./assets/image92.png)

**destroy** — deletes a user

![UserController@destroy deleting a user and returning 204](./assets/image-93.png)

---

## 7. Start the Development Server

```bash
php artisan serve
```

The API is now available at `http://127.0.0.1:8000`.

---

## 8. Verify Routes and Endpoints

Confirm all routes are registered correctly:

```bash
php artisan route:list
```

![Terminal output of route:list showing all /api/users/* routes mapped to UserController](./assets/image-23.png)

Test the `index` endpoint with curl:

```bash
curl http://127.0.0.1:8000/api/users
```

You should receive a JSON array of the seeded users. You can also import the base URL into Postman or Insomnia and test each endpoint (`GET`, `POST`, `PUT`, `DELETE`) against `/api/users`.
