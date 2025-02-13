Complete Guide: Setting Up Development Environment with Docker, Ionic, and PostgreSQL

# Complete Guide: Setting Up Development Environment with Docker, Ionic, and PostgreSQL

## Index
1. Introduction to Using Docker for Development
2. Setting Up Docker and Docker Compose
3. Creating the Ionic Project Inside the Container
4. Backend Configuration (Flask + PostgreSQL)
5. VS Code Configuration
6. Project Execution and Testing
7. PostgreSQL Database in Docker
8. Conclusion and Best Practices

## 1. Introduction to Using Docker for Development
### Why use Docker?
- **Standardization**: All developers use the same environment.
- **Avoids conflicts**: Dependency versions are kept consistent.
- **Ease of use**: No need to install everything on the operating system, just run the container.

By using containers for each service (frontend, backend, database), we ensure an isolated and efficient environment.

## 2. Setting Up Docker and Docker Compose
### Installing Docker
Download and install Docker from the official website:  
🔗 [Docker Desktop](https://www.docker.com/products/docker-desktop)

To check the installation, run:
```sh
docker --version
docker-compose --version
```
If the commands return versions, the installation was successful.

### Creating docker-compose.yml
The `docker-compose.yml` file defines our containers:
```yaml
version: "3.8"

services:
  frontend:
    build: ./frontend
    volumes:
      - ./frontend:/app
    ports:
      - "8100:8100"

  backend:
    build: ./backend
    depends_on:
      - db
    ports:
      - "5000:5000"
    environment:
      DATABASE_URL: postgresql://htmicron:htmicron@db:5432/mkt_database
    volumes:
      - ./backend:/app

  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: htmicron
      POSTGRES_PASSWORD: htmicron
      POSTGRES_DB: mkt_database
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

## 3. Creating the Ionic Project Inside the Container
After starting the containers, access the frontend container terminal and create the project:
```sh
docker-compose up -d
docker exec -it <container_id> bash
ionic start frontend-app blank --type=angular --standalone
```

### Standalone vs NgModule in Angular
- **Standalone**: More modern and modular structure.
- **NgModule**: Traditional Angular structure used in older projects.

We recommend using Standalone for new projects.

## 4. Backend Configuration (Flask + PostgreSQL)
### Creating the backend environment
Inside the project folder:
```sh
mkdir backend && cd backend
touch app.py Dockerfile requirements.txt
mkdir models
```

### Creating the backend Dockerfile
```Dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### Creating requirements.txt
```txt
flask
flask-sqlalchemy
flask-restful
flask-cors
psycopg2-binary
```

### Configuring Flask and PostgreSQL
Creating `app.py` with the API:
```python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_restful import Api, Resource
import os
from flask_cors import CORS

app = Flask(__name__)
api = Api(app)
CORS(app)

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://htmicron:htmicron@db:5432/mkt_database")
app.config["SQLALCHEMY_DATABASE_URI"] = DATABASE_URL
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)

class Purchase(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    item = db.Column(db.String(100), nullable=False)
    quantity = db.Column(db.Integer, nullable=False)
    price = db.Column(db.Float, nullable=False)

with app.app_context():
    db.create_all()

api.add_resource(Purchase, "/purchases", "/purchases/<int:purchase_id>")

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0")
```

## 5. VS Code Configuration
To edit files inside the container, use the Remote - Containers extension in VS Code.

## 6. Project Execution and Testing
To run the services:
```sh
docker-compose up --build
```
Access in the browser:
- Frontend: http://localhost:8100
- Backend: http://localhost:5000/purchases

## 7. PostgreSQL Database in Docker
### Accessing the database
```sh
docker exec -it <db_container_id> psql -U htmicron -d mkt_database
```
### Checking tables in PostgreSQL
```sql
SELECT * FROM purchases;
```

## 8. Conclusion and Best Practices
### Benefits of this method
- ✅ Identical development environment for all developers.
- ✅ Easy to launch and test the application quickly.
- ✅ Isolation between services, avoiding conflicts.

### Common errors and how to avoid them
- ❌ Error: Container exits immediately after starting  
  ✔ Solution: Check the CMD command in the Dockerfile. Flask needs to be running actively.

- ❌ Error: Frontend cannot access backend  
  ✔ Solution: Check if the backend is correctly exposed (http://localhost:5000).

### Future expansions
🚀 Suggested improvements:
- Implement authentication with JWT.
- Create a logging system for monitoring.
- Set up CI/CD for automatic deployment.

