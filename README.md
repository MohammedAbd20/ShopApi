# ShopApi

A RESTful API built with **Laravel** for managing shops, shop branches, product categories, items and customer comments. It includes JWT authentication, OTP account verification and security-question password reset.

![Laravel](https://img.shields.io/badge/Laravel-FF2D20?logo=laravel&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-777BB4?logo=php&logoColor=white)
![JWT](https://img.shields.io/badge/Auth-JWT-000000?logo=jsonwebtokens&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-Apache_2.0-blue)

## Features

- User registration and login with **JWT** (Bearer token)
- **OTP** verification and refresh
- Password reset through a security question
- CRUD for users, shops, shop branches, categories, items and item comments
- User addresses
- Filtering by status (`enabled` / `disabled`), search and `latest` endpoints
- Consistent JSON response envelope for success and error cases
- Docker setup with Nginx

## Architecture

```
Controller  →  Service  →  Model  →  Database
 (HTTP)      (business     (Eloquent)
              logic)
```

- **Controllers**: entry point for each API group
- **Services**: the logic behind each endpoint
- **Models**: the data saved in the database
- **JWT**: issued on login and sent as `Authorization: Bearer <token>`

## Getting started

### Requirements

- PHP 7.3+ (or 8.x) and Composer
- Laravel 8, with `tymon/jwt-auth` for tokens
- MySQL or MariaDB
- Docker (optional)

### Run locally

```bash
git clone https://github.com/MohammedAbd20/ShopApi.git
cd ShopApi
composer install
cp .env.example .env
php artisan key:generate
php artisan jwt:secret
php artisan migrate
php artisan serve
```

The API is then available at `http://127.0.0.1:8000/api/v1`.

### Run with Docker

```bash
docker compose up -d --build
```

## API overview

Base URL: `/api/v1`

| Group | Endpoints (examples) |
|---|---|
| **Users** | `POST /users`, `POST /users/login`, `POST /users/otp/verify`, `POST /users/otp/refresh`, `GET /users`, `GET /users/{id}`, `GET /users/enabled`, `GET /users/disabled`, `DELETE /users/{id}` |
| **Password reset** | `GET /users/reset/question`, `POST /users/reset/answer` |
| **Addresses** | `POST /shops/users/addresses`, `GET /shops/users/addresses`, `GET /shops/users/addresses/{id}` |
| **Shops** | `POST /shops`, `GET /shops`, `GET /shops/{id}`, `GET /shops/{id}/items`, `GET /shops/{id}/menu`, `GET /shops/search`, `GET /shops/latest`, `DELETE /shops/{id}` |
| **Branches** | `POST /shops/branches`, `GET /shops/branches`, `GET /shops/branches/{id}`, `DELETE /shops/branches/{id}` |
| **Categories** | `POST /categories`, `GET /categories`, `GET /categories/{id}`, `DELETE /categories/{id}` |
| **Items** | `POST /items`, `GET /items`, `GET /items/{id}`, `GET /items/search`, `GET /items/latest`, `DELETE /items/{id}` |
| **Comments** | `POST /items/comments`, `GET /items/comments`, `GET /items/comments/{id}` |

Most list groups also support `/enabled` and `/disabled`.

## Examples

### Create a user

`POST /api/v1/users`

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "********",
  "gender": "Female",
  "age": 28,
  "phone_number": "0000000000",
  "location_name": "Gaza",
  "type": "Customer",
  "security_question": "Your first school?",
  "security_question_answer": "********"
}
```

### Login

`POST /api/v1/users/login`

```json
{ "email": "jane@example.com", "password": "********" }
```

Response:

```json
{
  "code": 200,
  "message": "Data Found",
  "status": true,
  "data": {
    "user": { "id": 7, "name": "Jane Doe", "email": "jane@example.com" },
    "token": "<jwt-token>",
    "tokenKey": "Bearer"
  }
}
```

### Error format

```json
{
  "code": 400,
  "message": "Failed",
  "status": false,
  "errorMessage": "Incorrect Email Or Password Please Try Again"
}
```

## Tests

```bash
php artisan test
```

## License

Released under the [Apache 2.0 License](LICENSE).

## Author

**Mohammed Abd Elrahman**: [GitHub](https://github.com/MohammedAbd20) · [LinkedIn](https://www.linkedin.com/in/mohammed-abd-elrahman-9618a732b/)
