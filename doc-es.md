# Laravel REST API — Guía de Configuración

Configuración de la API REST basada en Laravel desde cero. La API expone un recurso `users` y usa Laravel Sanctum para autenticación basada en tokens. Stack: **Laravel 11**, **Sanctum**, **MySQL 8**.

---

## Requisitos previos

Antes de comenzar, asegúrate de tener instalado lo siguiente:

- PHP >= 8.2
- Composer >= 2.x
- MySQL >= 8.0
- Un cliente de base de datos (TablePlus, DBeaver o MySQL Workbench)
- (Opcional) Postman o Insomnia para probar los endpoints

---

## 1. Configuración del proyecto

Crea el proyecto Laravel y ábrelo en tu editor:

```bash
composer create-project laravel/laravel api
cd api
code .
```

Instala el scaffold de API. Esto publica la configuración de Sanctum y crea el archivo `routes/api.php`:

```bash
php artisan install:api
```

Después de ejecutar este comando, Laravel registra el middleware `sanctum` y crea `routes/api.php`.

![Sanctum instalado — salida confirmando que el scaffold de API fue publicado](./assets/image.png)
![routes/api.php creado con la ruta de token de Sanctum por defecto](./assets/image-1.png)

---

## 2. Conexión a la base de datos

Abre `.env` en la raíz del proyecto. Localiza el bloque `DB_*` (líneas 24–28) y completa con tus credenciales locales:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nombre_de_tu_base_de_datos
DB_USERNAME=tu_usuario
DB_PASSWORD=tu_contraseña
```

![Archivo .env con el bloque DB completado con credenciales locales](./assets/image-4.png)

### Crear la base de datos

Antes de ejecutar las migraciones, crea una base de datos vacía en MySQL con el nombre que definiste en `DB_DATABASE`. Usa tu cliente de base de datos o ejecuta:

```sql
CREATE DATABASE nombre_de_tu_base_de_datos;
```

![Base de datos creada en el cliente — sin tablas aún](./assets/image-5.png)
![Cliente MySQL mostrando la nueva base de datos seleccionada](./assets/image-6.png)
![Vista del esquema confirmando que la base de datos está vacía antes de migrar](./assets/image-7.png)

---

## 3. Migraciones

Laravel incluye migraciones por defecto para `users`, `password_reset_tokens`, `sessions` y `cache`. Ejecútalas para crear el esquema:

```bash
# Estado antes de migrar — no existen tablas aún
```

![Cliente de base de datos mostrando que no hay tablas antes de migrar](./assets/image-9.png)

```bash
php artisan migrate
```

![Salida del terminal confirmando que todas las migraciones se ejecutaron exitosamente](./assets/image-10.png)
![Cliente de base de datos mostrando las tablas por defecto de Laravel creadas](./assets/image-11.png)

---

## 4. Seeders — Poblar la tabla de usuarios

El `DatabaseSeeder` usa un factory para generar usuarios falsos. Abre `database/seeders/DatabaseSeeder.php` y realiza los siguientes cambios:

1. Elimina el bloque de código de ejemplo (resaltado en la captura de pantalla).
2. Descomenta la llamada a `User::factory()` y define la cantidad de registros a generar.

```php
// Ejemplo: generar 10 usuarios falsos
User::factory(10)->create();
```

![DatabaseSeeder antes de editar — el código de ejemplo a eliminar está resaltado](./assets/image-2.png)
![DatabaseSeeder después de editar — la llamada al factory activa con la cantidad definida](./assets/image-3.png)

Ejecuta el seeder:

```bash
# Tabla de usuarios antes del seeder — vacía
```

![Tabla de usuarios vacía antes de ejecutar el seeder](./assets/image-12.png)

```bash
php artisan db:seed
```

![Salida del terminal mostrando el seeder ejecutándose y generando registros](./assets/image-13.png)
![Tabla de usuarios después del seeder — 10 usuarios falsos insertados](./assets/image-14.png)

---

## 5. Controlador de recursos

Genera un controlador de recursos para el modelo `User`. El flag `-r` genera los siete métodos RESTful estándar (`index`, `show`, `store`, `update`, `destroy`, etc.):

```bash
php artisan make:controller Api/UserController -r
```

Abre `app/Http/Controllers/Api/UserController.php` e inyecta el modelo `User` mediante route-model binding en `show`, `update` y `destroy`.

![UserController generado con los stubs de métodos vacíos por defecto](./assets/image-21.png)
![UserController después de inyectar el modelo User](./assets/image-22.png)

---

## 6. Rutas de la API

Abre `routes/api.php`. Por defecto solo contiene la ruta de token de Sanctum. Registra el recurso de usuarios:

![routes/api.php en estado por defecto — solo la ruta de token de Sanctum](./assets/image-18.png)

Agrega el siguiente import y el registro de la ruta:

```php
use App\Http\Controllers\Api\UserController;

Route::apiResource('users', UserController::class);
```

![routes/api.php después de agregar la ruta apiResource para UserController](./assets/image-19.png)

Cada método del controlador corresponde a un endpoint HTTP estándar. A continuación las implementaciones:

**index** — retorna la lista de usuarios

![UserController@index retornando todos los usuarios como JSON](./assets/image-20.png)

**store** — crea un nuevo usuario

![UserController@store validando y persistiendo un nuevo usuario](./assets/image-90.png)

**show** — retorna un usuario por ID

![UserController@show retornando un usuario mediante route-model binding](./assets/image91.png)

**update** — actualiza un usuario existente

![UserController@update aplicando cambios validados a un usuario](./assets/image92.png)

**destroy** — elimina un usuario

![UserController@destroy eliminando un usuario y retornando 204](./assets/image-93.png)

---

## 7. Iniciar el servidor de desarrollo

```bash
php artisan serve
```

La API estará disponible en `http://127.0.0.1:8000`.

---

## 8. Verificar rutas y endpoints

Confirma que todas las rutas están registradas correctamente:

```bash
php artisan route:list
```

![Salida del terminal de route:list mostrando todas las rutas /api/users/* mapeadas a UserController](./assets/image-23.png)

Prueba el endpoint `index` con curl:

```bash
curl http://127.0.0.1:8000/api/users
```

Deberías recibir un arreglo JSON con los usuarios generados por el seeder. También puedes importar la URL base en Postman o Insomnia y probar cada endpoint (`GET`, `POST`, `PUT`, `DELETE`) contra `/api/users`.
