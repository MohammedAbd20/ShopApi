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

Every request and response below was captured from a local run of this API (SQLite, `php artisan serve`). Timestamps and the token are shortened, and `stackTrace` is left out of the error responses (see the note at the end).

All requests send `Accept: application/json` and `Content-Type: application/json`.

### 1. Register

`POST /api/v1/users`

```json
{
  "name": "Sara Ahmad",
  "image": "https://example.com/sara.png",
  "email": "sara@example.com",
  "password": "Secret123",
  "gender": "Female",
  "age": 28,
  "phone_number": "+970591234567",
  "location_lat": 31.5,
  "location_lng": 34.46,
  "location_name": "Gaza City",
  "security_question": "First school?",
  "security_question_answer": "Al-Zaytoun",
  "type": "User"
}
```

`gender` is `Male` or `Female`. `type` is `User`, `Admin` or `Shop`. The password must be 8 to 25 characters.

Response, HTTP 201:

```json
{
  "code": 201,
  "message": "Data Found",
  "status": true,
  "path": "http://127.0.0.1:8200/api/v1/users",
  "data": {
    "name": "Sara Ahmad",
    "email": "sara@example.com",
    "gender": "Female",
    "age": 28,
    "phone_number": "+970591234567",
    "location_name": "Gaza City",
    "type": "User",
    "is_account_activated": 0,
    "status": "NOT_VERIFIED",
    "id": 1
  }
}
```

A new account cannot log in until its phone number is verified:

```json
{ "code": 400, "message": "Failed", "status": false,
  "errorMessage": "Account Not Verified, Please Verify The Phone Number" }
```

### 2. Verify the phone number

`POST /api/v1/users/otp/refresh` with `{ "id": 1 }` creates a verification code and answers `"message": "Code Sent"`. The code is stored in the `phone_numbers` table, and no SMS provider is configured in this repository, so read it from there when running locally.

`POST /api/v1/users/otp/verify`

```json
{ "id": 1, "code": "<code from phone_numbers>" }
```

The response is the same user object with `"status": "VERIFIED"` and `"is_account_activated": 1`.

### 3. Log in

`POST /api/v1/users/login`

```json
{ "email": "sara@example.com", "password": "Secret123" }
```

```json
{
  "code": 200,
  "message": "Data Found",
  "status": true,
  "data": {
    "user": { "id": 1, "name": "Sara Ahmad", "email": "sara@example.com", "status": "VERIFIED" },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "tokenKey": "Bearer"
  }
}
```

### 4. Create and list categories

`POST /api/v1/categories`

```json
{ "nameAr": "مطاعم", "nameEn": "Restaurants", "image": "https://example.com/restaurants.png" }
```

```json
{
  "code": 201,
  "message": "Data Found",
  "status": true,
  "data": {
    "nameEn": "Restaurants",
    "image": "https://example.com/restaurants.png",
    "is_enabled": 1,
    "id": 1
  }
}
```

`GET /api/v1/categories`

```json
{
  "code": 200,
  "message": "Data Found",
  "status": true,
  "data": [
    { "nameEn": "Restaurants", "image": "https://example.com/restaurants.png", "is_enabled": 1, "id": 1 }
  ],
  "size": 1
}
```

### Errors

Failures return HTTP 400 and the same envelope with `status: false` and an `errorMessage`:

| Request | `errorMessage` |
|---|---|
| Register with a password shorter than 8 characters | `User Password Length Required Min 8 (Field password)` |
| Register with an email that already exists | `Email Already Used ...` |
| Log in with a wrong password | `Incorrect Email Or Password Please Try Again` |
| Create a category without `nameEn` | `English Name Required` |

> **Known issue:** error responses also include a `stackTrace` field with server file paths. Return it only when `APP_DEBUG` is on before deploying this API anywhere public.

## Tests

```bash
php artisan test
```

## License

Released under the [Apache 2.0 License](LICENSE).

## Author

**Mohammed Hamdi Abdelrahman**: [GitHub](https://github.com/MohammedAbd20) · [LinkedIn](https://www.linkedin.com/in/mohammed-abdelrahman-web/)
