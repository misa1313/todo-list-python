# FastAPI Application

A simple To-Do list application built with FastAPI, allowing users to create, read, update, and delete tasks.

## Requirements

* Docker
* Docker-compose
* Python 3.11
* Poetry (python-poetry)

## Features

- User authentication (login and registration)
- Create, read, update, and delete tasks
- Assign tasks to users
- Set task completion status
- Role-based access control (admin and user roles)
- Daily reminders for incomplete tasks

## Setup

### Step 1: Configure ENV variables

Create a file ".env" with the following variables:

```bash
app_url=localhost (the app domain, optional and defaults to localhost)

db_host=HOST
db_database=DATABASE
db_user=DB_USER
db_password=USER_PASSWORD

test_db_database=TEST_DATABASE
test_db_user=TEST_DB_USER
test_db_password=TEST_USER_PASSWORD

admin_username=ADMIN_USERNAME (Optional)
admin_email=ADMIN_USER_EMAIL (Optional)
admin_password=ADMIN_USER_PASSWORD (Optional)  
secret_key = "random string"
```
A test database will be created as well. If admin username, email and password are not provided, default values will be used. The secret key can be generated by running: 
```bash
openssl rand -hex 32  
```

### Step 2: Start the containers

Start the containers by running:

```bash
docker-compose up -d
```
After the databases are up, the app will be started. If any of the tests fails, the app's container will be stopped.
The app is accessible on port 8000.


### Endpoints

#### Miscellaneous Routes

| Method | Endpoint              | Description           | Authentication Required |
|--------|-----------------------|-----------------------|-------------------------|
| `GET`  | `/api/v1/`            | Root Test             | No                      |
| `GET`  | `/api/v1/test`        | Test Route            | No                      |
| `GET`  | `/api/v1/db-connection` | Testing Connection  | No                      |
| `GET`  | `/api/v1/schema`      | Get Schema Version    | No                      |

#### Tasks

| Method  | Endpoint                   | Description        | Authentication Required |
|---------|----------------------------|--------------------|-------------------------|
| `GET`   | `/api/v1/tasks/`           | Get All Todos      | Yes                     |
| `POST`  | `/api/v1/tasks/`           | Add Todo           | Yes                     |
| `GET`   | `/api/v1/tasks/{task_id}`  | Get Task by Id     | Yes                     |
| `PUT`   | `/api/v1/tasks/{task_id}`  | Update Todo        | Yes                     |
| `DELETE`| `/api/v1/tasks/{task_id}`  | Delete Todo        | Yes (Owner or Admin)    |
| `PUT`   | `/api/v1/tasks/{task_id}/finish` | Mark Completed | Yes                   |

#### Users

| Method  | Endpoint                       | Description               | Authentication Required |
|---------|--------------------------------|---------------------------|-------------------------|
| `POST`  | `/api/v1/users/register`       | Create User               | No                      |
| `GET`   | `/api/v1/users/{id_}`          | Get User                  | No                      |
| `PUT`   | `/api/v1/users/{id_}`          | Update User               | Yes (Owner or Admin)    |
| `DELETE`| `/api/v1/users/{id_}`          | Delete User               | Yes (Owner or Admin)    |
| `PATCH` | `/api/v1/users/{id_}`          | Set Role                  | Yes (Admin only)        |
| `GET`   | `/api/v1/users/`               | Get All Users             | Yes (Admin only)        |
| `GET`   | `/api/v1/users/reminders`  | Configure daily reminders | Yes (Owner or Admin)    |
| `GET`   | `/api/v1/users/send_reminders` | Send daily reminders      | Yes (Admin only)        |

#### Authentication

| Method  | Endpoint                   | Description        | Authentication Required |
|---------|----------------------------|--------------------|-------------------------|
| `POST`  | `/api/v1/login`            | Login              | No                      |

## Authentication

To access endpoints that require authentication, you need to include a valid JWT token in the `Authorization` header as a Bearer token. The "login" endpoint generates a new token with a default expiration of 7 days. 

Example of a user login request using curl:
```bash
curl -i localhost:8000/api/v1/login -XPOST -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=USERNAME&password=PASSWORD' -w '\n'
```
The above will generate a new token. 

## Daily Reminders

The daily reminder feature sends an automated email to users who have the `daily_reminder` option enabled. It aggregates all incomplete tasks for each user and sends a summary in the email body. The reminders are triggered via a background scheduler that runs at a configured interval. The scheduler calls the `/send_reminders` API, which queries the database for users with pending tasks and sends the appropriate emails.

## Sample Requests (Curl)

Register a new user:
```bash
curl -i localhost:8000/api/v1/users/register -XPOST -H 'Content-Type: application/json' -d '{"username": "user", "email": "test@test.com", "password": "password"}' -w '\n'
```

Update user - in this case, the email of user with ID 2:
```bash
curl -i localhost:8000/api/v1/users/2 -XPUT -H 'Content-Type: application/json' -d '{"email":"test2@test.com"}' -H "Authorization: Bearer $token" -w '\n'
```

Delete user with ID 2:
```bash
curl -i localhost:8000/api/v1/users/2 -XDELETE -H 'Content-Type: application/json' -H "Authorization: Bearer $token" -w '\n'
```

Enable the daily reminders:
```bash
curl -i localhost:8000/api/v1/users/reminders -XPOST -H 'Content-Type: application/json' -H "Authorization: Bearer $token" -d '{"reminder":"yes"}' -w '\n'
```

Get all tasks:
```bash
curl -i localhost:8000/api/v1/tasks/ -XGET -H 'Content-Type: application/json' -H "Authorization: Bearer $token" -w '\n'
```

Add a new task:
```bash
curl -i localhost:8000/api/v1/tasks/ -XPOST  -H 'Content-Type: application/json' -H "Authorization: Bearer $token" -d '{"title":"Title", "description":"Description", "is_finished": "False"}' -w '\n'
```

Mark task as completed:
```bash
curl -i localhost:8000/api/v1/tasks/1/finish -XPUT -H 'Content-Type: application/json' -H "Authorization: Bearer $token" -d '{"is_finished": "True"}' -w '\n'
```


## Contributions

Unit tests were added to the project. Also, linter checks might be performed by following the instructions provided below.


### Running Linter Checks

To run linter checks, follow these steps:

1.  **Install dependencies**:

```bash
poetry install --with dev
```

2.  **Run `pylint`**:

```bash
poetry shell
poetry run pylint *.py **/*.py
```
