Task Management API
A simple Task Management REST API built with Python Flask and MySQL, then containerized using Docker.

The project demonstrates:

Flask REST API development
MySQL database integration
Python virtual environment setup
Docker containers
Docker networking
Persistent Docker volumes
CRUD operations
Database health checking
Environment variable configuration
🛠️ Technologies Used
Technology	Version
Python	3.12
Flask	3.1.2
MySQL	8.0
mysql-connector-python	9.4.0
python-dotenv	1.1.1
Docker	Latest
📂 Project Structure
NSDC lab/
│
├── app.py
├── database.py
├── init.sql
├── requirements.txt
├── Dockerfile
├── .dockerignore
├── .gitignore
└── .env
Note: .env, venv/, __pycache__/ and .pyc files should not be committed to Git.

🚀 Stage 1: Run the Application Locally
1. Start MySQL using Docker
Pull the MySQL 8.0 image:

docker pull mysql:8.0
Create the MySQL container:

docker run -d ^
  --name task-mysql ^
  -e MYSQL_ROOT_PASSWORD=root123 ^
  -e MYSQL_DATABASE=taskdb ^
  -e MYSQL_USER=appuser ^
  -e MYSQL_PASSWORD=app123 ^
  -p 3306:3306 ^
  mysql:8.0
Check the container:

docker ps
Check the MySQL logs:

docker logs task-mysql
Wait until the logs show something similar to:

ready for connections
2. Create the Database Table
Create a file named:

init.sql
Add:

USE taskdb;

CREATE TABLE IF NOT EXISTS tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    completed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Copy the SQL file into the MySQL container:

docker cp init.sql task-mysql:/init.sql
Execute the SQL:

docker exec -i task-mysql mysql -uappuser -papp123 taskdb < init.sql
A warning may be displayed during this command. It can be ignored for this lab setup.

Verify the table:

docker exec -it task-mysql mysql -uappuser -papp123 taskdb
Inside MySQL:

SHOW TABLES;

DESCRIBE tasks;

exit;
🐍 3. Create Python Virtual Environment
Create the virtual environment:

python -m venv venv
Activate it on Windows:

venv\Scripts\activate
You should see:

(venv)
at the beginning of your PowerShell prompt.

📦 4. Install Dependencies
Create:

requirements.txt
Add:

Flask==3.1.2
mysql-connector-python==9.4.0
python-dotenv==1.1.1
Install the dependencies:

pip install -r requirements.txt
🔐 5. Configure Environment Variables
Create:

.env
Add:

DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=appuser
DB_PASSWORD=app123
DB_NAME=taskdb
🗄️ 6. Database Connection
Create:

database.py
import os
import mysql.connector
from dotenv import load_dotenv

load_dotenv()


def get_connection():
    return mysql.connector.connect(
        host=os.getenv("DB_HOST"),
        port=int(os.getenv("DB_PORT", 3306)),
        user=os.getenv("DB_USER"),
        password=os.getenv("DB_PASSWORD"),
        database=os.getenv("DB_NAME")
    )
This file has one main responsibility:

Create a connection between Python and MySQL.

🌐 7. Flask Application
Create:

app.py
The application provides the following endpoints:

Method	Endpoint	Description
GET	/	Check if API is running
GET	/health	Check database connection
GET	/tasks	Get all tasks
GET	/tasks/<id>	Get a single task
POST	/tasks	Create a task
PUT	/tasks/<id>	Update a task
DELETE	/tasks/<id>	Delete a task
Run the application:

python app.py
The application will run on:

http://127.0.0.1:5000
❤️ 8. Health Check
Open:

http://localhost:5000/health
A successful response will look similar to:

{
    "status": "healthy",
    "database": "connected"
}
🔌 9. API Usage
Create a Task
POST http://localhost:5000/tasks
Request body:

{
    "title": "Learn Kubernetes",
    "description": "Move this application to Kubernetes"
}
Get All Tasks
GET http://localhost:5000/tasks
Get a Single Task
GET http://localhost:5000/tasks/1
Update a Task
PUT http://localhost:5000/tasks/1
Request body:

{
    "title": "Learn Kubernetes",
    "description": "Learn Pods and Services",
    "completed": true
}
Delete a Task
DELETE http://localhost:5000/tasks/1
🐳 Stage 2: Dockerize the Python Application
The application can be fully containerized using Docker.

The final architecture uses:

Python Flask container
MySQL container
Docker network
Persistent MySQL volume
1. Stop the Existing MySQL Container
docker stop task-mysql
Remove it:

docker rm task-mysql
🌐 2. Create Docker Network
Create:

docker network create task-network
Verify:

docker network ls
Architecture:

task-network
│
├── Python Container
│
└── MySQL Container
💾 3. Create Persistent MySQL Volume
Create:

docker volume create task-mysql-data
Verify:

docker volume ls
The volume ensures that the database data survives container recreation.

MySQL Container
       │
       ▼
task-mysql-data
       │
       ▼
Persistent Database
🐬 4. Start MySQL with Network and Volume
docker run -d ^
  --name task-mysql ^
  --network task-network ^
  -e MYSQL_ROOT_PASSWORD=root123 ^
  -e MYSQL_DATABASE=taskdb ^
  -e MYSQL_USER=appuser ^
  -e MYSQL_PASSWORD=app123 ^
  -v task-mysql-data:/var/lib/mysql ^
  mysql:8.0
Check:

docker ps
🗃️ 5. Create the Tasks Table Again
Execute:

docker exec -i task-mysql mysql -uappuser -papp123 taskdb < init.sql
Verify:

docker exec -it task-mysql mysql -uappuser -papp123 taskdb
Inside MySQL:

SHOW TABLES;

exit;
🐍 6. Create Dockerfile
Create a file named:

Dockerfile
Add:

FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .
COPY database.py .

EXPOSE 5000

CMD ["python", "app.py"]
Build the Docker image:

docker build -t task-python-app .
Verify:

docker images
You should see:

task-python-app
▶️ 7. Run the Python Container
Run:

docker run -d ^
  --name task-python ^
  --network task-network ^
  -p 5000:5000 ^
  -e DB_HOST=task-mysql ^
  -e DB_PORT=3306 ^
  -e DB_USER=appuser ^
  -e DB_PASSWORD=app123 ^
  -e DB_NAME=taskdb ^
  task-python-app
🔗 Understanding DB_HOST
The most important part of the Docker configuration is:

DB_HOST=task-mysql
Both containers are connected to:

task-network
Docker provides internal DNS, so the hostname:

task-mysql
automatically resolves to the MySQL container.

Therefore:

Python Container
      │
      │ task-mysql:3306
      ▼
MySQL Container
We do not need to know the MySQL container's IP address.

Also, inside the Python container we do not use:

localhost
for the MySQL connection.

🔍 8. Check Both Containers
Run:

docker ps
You should see both:

task-python
task-mysql
Architecture:

Docker
│
└── task-network
    │
    ├── task-python
    │      │
    │      └── Flask :5000
    │
    └── task-mysql
           │
           └── MySQL :3306
📜 9. Check Python Logs
Run:

docker logs task-python
You should see something similar to:

Running on all addresses (0.0.0.0)
Running on http://127.0.0.1:5000
🌍 10. Test the Application
Open in your browser:

http://localhost:5000
Expected response:

{
    "message": "Task Management API is running"
}
The port mapping is:

5000 → 5000
Browser request flow:

Browser
   │
   │ localhost:5000
   ▼
Windows
   │
   │ port mapping
   ▼
Python Container
❤️ 11. Test Database Connection
Open:

http://localhost:5000/health
Request flow:

Browser
   │
   │ localhost:5000
   ▼
Python Container
   │
   │ task-mysql:3306
   ▼
MySQL Container
   │
   ▼
Docker Volume
🧪 12. Test Docker Networking
Run:

docker network inspect task-network
This allows you to verify that both containers are connected to the same Docker network.

💾 13. Test Database Persistence
Stop the MySQL container:

docker stop task-mysql
Remove it:

docker rm task-mysql
Recreate it using the same volume:

docker run -d ^
  --name task-mysql ^
  --network task-network ^
  -e MYSQL_ROOT_PASSWORD=root123 ^
  -e MYSQL_DATABASE=taskdb ^
  -e MYSQL_USER=appuser ^
  -e MYSQL_PASSWORD=app123 ^
  -v task-mysql-data:/var/lib/mysql ^
  mysql:8.0
Connect:

docker exec -it task-mysql mysql -uappuser -papp123 taskdb
Run:

SELECT * FROM tasks;
The previously stored tasks should still exist.

This demonstrates Docker volume persistence.

🧹 14. Improved Dockerfile
A cleaner Dockerfile can be used:

FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .
COPY database.py .

EXPOSE 5000

CMD ["python", "app.py"]
Why these environment variables?
PYTHONDONTWRITEBYTECODE=1

Prevents unnecessary .pyc files.

PYTHONUNBUFFERED=1

Makes Python logs appear immediately, which is useful when working with Docker and CI/CD tools such as Jenkins.

🏗️ Final Architecture
                    Docker
┌─────────────────────────────────────────┐
│                                         │
│   Browser                               │
│      │                                  │
│      │ localhost:5000                   │
│      ▼                                  │
│  ┌──────────────────────┐               │
│  │     task-python      │               │
│  │                      │               │
│  │   Flask / Python     │               │
│  │       :5000          │               │
│  └──────────┬───────────┘               │
│             │                           │
│             │ task-network              │
│             ▼                           │
│  ┌──────────────────────┐               │
│  │      task-mysql      │               │
│  │                      │               │
│  │       MySQL          │               │
│  │        :3306         │               │
│  └──────────┬───────────┘               │
│             │                           │
│             ▼                           │
│      task-mysql-data                    │
│       Docker Volume                     │
│                                         │
└─────────────────────────────────────────┘
✅ Project Checklist
Local Setup
 Docker is running
 MySQL container is running
 taskdb exists
 tasks table exists
 Python virtual environment works
 Flask starts
 / endpoint works
 /health shows database connected
 POST /tasks works
 GET /tasks works
 GET /tasks/<id> works
 PUT /tasks/<id> works
 DELETE /tasks/<id> works
 Data is visible inside MySQL
 .env is excluded from Git
Docker Setup
 Docker network task-network created
 Docker volume task-mysql-data created
 MySQL container running
 Python Dockerfile created
 Python Docker image built
 Python container running
 Both containers are on task-network
 localhost:5000 works
 /health shows database connected
 POST /tasks works
 GET /tasks works
 PUT /tasks/<id> works
 DELETE /tasks/<id> works
 Python can resolve task-mysql
 MySQL data survives container recreation
 .dockerignore created
 .env is not inside the Docker image
📌 Key Concepts Demonstrated
This project demonstrates the following practical concepts:

Python + Flask

REST API development
CRUD operations
Database connection handling
MySQL

Database creation
Table creation
Insert, Read, Update and Delete operations
Docker

Docker images
Docker containers
Port mapping
Docker networks
Docker volumes
Container-to-container communication
Persistent storage
Environment Configuration

.env variables
Docker environment variables
Secure separation of configuration from source code
👨‍💻 Author
Aditya Devidas Sonawale

Task Management API — Flask + MySQL + Docker
