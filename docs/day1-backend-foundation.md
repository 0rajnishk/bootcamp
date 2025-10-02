---
layout: default
title: Day 1 - Backend Foundation - Flask API & Data Modeling
nav_order: 2
---



# Day 1: Backend Foundation - Flask API & Data Modeling

## Learning Objectives
By the end of this day, you will be able to:
- Set up a Flask development environment
- Create RESTful API endpoints
- Define data models using SQLAlchemy
- Perform basic CRUD operations on a database
- Test APIs using Postman
- Return JSON responses from Flask routes



## Environment Setup

### 1. Create Virtual Environment
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

### 2. Install Dependencies
```bash
pip install flask sqlalchemy flask-sqlalchemy flask-cors
```

### 3. Install Postman
Download and install Postman from [postman.com](https://www.postman.com/downloads/) for API testing.

## Flask Basics

### What is Flask?
Flask is a lightweight web framework for Python. It's perfect for beginners because it's simple but powerful.

### Creating Your First Flask API

```python
from flask import Flask, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # Enable CORS for API testing

@app.route('/')
def hello():
    return jsonify({"message": "Hello, World!", "status": "success"})

@app.route('/api/health')
def health_check():
    return jsonify({"status": "healthy", "service": "Task Manager API"})

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

### Understanding API Routes
- `@app.route('/')` - Decorator that tells Flask what URL should trigger our function
- `def hello():` - Function that runs when someone visits the URL
- `jsonify()` - Converts Python dictionaries to JSON responses
- `CORS(app)` - Enables Cross-Origin Resource Sharing for API testing

## Data Modeling with SQLAlchemy

### What is SQLAlchemy?
SQLAlchemy is an Object-Relational Mapping (ORM) library. It lets you work with databases using Python objects instead of writing SQL directly.

### Creating a Data Model

```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    description = db.Column(db.Text, nullable=True)
    completed = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    def __repr__(self):
        return f'<Task {self.title}>'
```

### Understanding the Model
- `id` - Primary key (unique identifier)
- `title` - Task title (required, max 100 characters)
- `description` - Optional task description
- `completed` - Boolean flag (defaults to False)
- `created_at` - Timestamp when task was created

## Database Operations

### Setting Up the Database
```python
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy
from flask_cors import CORS

app = Flask(__name__)
CORS(app)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///tasks.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

# Create all tables
with app.app_context():
    db.create_all()
```

### CRUD Operations

#### Create (Add new task)
```python
@app.route('/api/tasks', methods=['POST'])
def add_task():
    try:
        data = request.get_json()
        title = data.get('title')
        description = data.get('description', '')
        
        if not title:
            return jsonify({"error": "Title is required"}), 400
        
        new_task = Task(title=title, description=description)
        db.session.add(new_task)
        db.session.commit()
        
        return jsonify({
            "message": "Task created successfully",
            "task": {
                "id": new_task.id,
                "title": new_task.title,
                "description": new_task.description,
                "completed": new_task.completed,
                "created_at": new_task.created_at.isoformat()
            }
        }), 201
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

#### Read (Get all tasks)
```python
@app.route('/api/tasks', methods=['GET'])
def get_tasks():
    try:
        tasks = Task.query.all()
        tasks_list = []
        for task in tasks:
            tasks_list.append({
                "id": task.id,
                "title": task.title,
                "description": task.description,
                "completed": task.completed,
                "created_at": task.created_at.isoformat()
            })
        
        return jsonify({
            "tasks": tasks_list,
            "count": len(tasks_list)
        }), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

#### Update (Mark task as complete)
```python
@app.route('/api/tasks/<int:task_id>/complete', methods=['PUT'])
def complete_task(task_id):
    try:
        task = Task.query.get_or_404(task_id)
        task.completed = True
        db.session.commit()
        
        return jsonify({
            "message": "Task marked as complete",
            "task": {
                "id": task.id,
                "title": task.title,
                "completed": task.completed
            }
        }), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

#### Delete (Remove task)
```python
@app.route('/api/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    try:
        task = Task.query.get_or_404(task_id)
        task_title = task.title
        db.session.delete(task)
        db.session.commit()
        
        return jsonify({
            "message": f"Task '{task_title}' deleted successfully"
        }), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

## Testing APIs with Postman

### What is Postman?
Postman is a powerful tool for testing APIs. It allows you to send HTTP requests and view responses without writing frontend code.

### Setting Up Postman
1. Download and install Postman
2. Create a new workspace for your project
3. Set up environment variables for easy testing

### Testing Your API Endpoints

#### 1. Test Health Check
- **Method**: GET
- **URL**: `http://localhost:5000/api/health`
- **Expected Response**:
```json
{
    "status": "healthy",
    "service": "Task Manager API"
}
```

#### 2. Create a New Task
- **Method**: POST
- **URL**: `http://localhost:5000/api/tasks`
- **Headers**: `Content-Type: application/json`
- **Body** (raw JSON):
```json
{
    "title": "Learn Flask API",
    "description": "Complete the backend foundation course"
}
```

#### 3. Get All Tasks
- **Method**: GET
- **URL**: `http://localhost:5000/api/tasks`
- **Expected Response**:
```json
{
    "tasks": [
        {
            "id": 1,
            "title": "Learn Flask API",
            "description": "Complete the backend foundation course",
            "completed": false,
            "created_at": "2024-01-15T10:30:00"
        }
    ],
    "count": 1
}
```

#### 4. Mark Task as Complete
- **Method**: PUT
- **URL**: `http://localhost:5000/api/tasks/1/complete`

#### 5. Delete a Task
- **Method**: DELETE
- **URL**: `http://localhost:5000/api/tasks/1`

## Hands-On Exercise

### Exercise 1: Basic Flask API
1. Create a new Flask app with CORS enabled
2. Add a route `/api/health` that returns a JSON health check
3. Add a route `/api/info` that returns app information
4. Run the app and test both routes in Postman

### Exercise 2: Task Model & Database
1. Create a Task model with the fields shown above
2. Set up the database connection with SQLite
3. Create the database tables
4. Test database creation by running the app

### Exercise 3: CRUD API Operations
1. Create API routes for adding, viewing, completing, and deleting tasks
2. All routes should return JSON responses
3. Test all operations using Postman:
   - POST `/api/tasks` - Create a task
   - GET `/api/tasks` - Get all tasks
   - PUT `/api/tasks/{id}/complete` - Mark task complete
   - DELETE `/api/tasks/{id}` - Delete a task

### Exercise 4: Error Handling
1. Add proper error handling to all routes
2. Return appropriate HTTP status codes
3. Test error scenarios in Postman (invalid data, missing tasks, etc.)

## Key Concepts Summary

### Flask API Concepts
- **Routes**: URL patterns that trigger functions
- **JSON responses**: Structured data format for APIs
- **HTTP methods**: GET, POST, PUT, DELETE for different operations
- **CORS**: Cross-Origin Resource Sharing for API access
- **Error handling**: Proper HTTP status codes and error messages

### Database Concepts
- **Models**: Python classes representing database tables
- **ORM**: Object-Relational Mapping for database operations
- **CRUD**: Create, Read, Update, Delete operations
- **Sessions**: Database transaction management

### API Testing Concepts
- **Postman**: Tool for testing API endpoints
- **HTTP status codes**: 200 (success), 201 (created), 400 (bad request), 500 (server error)
- **JSON format**: Structured data exchange between client and server

### Best Practices
- Always use virtual environments
- Keep models in separate files
- Use meaningful variable names
- Handle errors gracefully with proper HTTP status codes
- Return consistent JSON response format
- Use debug mode only in development

## Common Issues and Solutions

### Issue: Database not found
**Solution**: Make sure to run `db.create_all()` after defining models

### Issue: Import errors
**Solution**: Check that all required packages are installed in your virtual environment

### Issue: CORS errors in Postman
**Solution**: Make sure to add `CORS(app)` to your Flask application

### Issue: JSON parsing errors
**Solution**: Ensure you're sending `Content-Type: application/json` header in Postman

### Issue: 404 errors in Postman
**Solution**: Check that your Flask app is running and the URL is correct

## Next Steps
Tomorrow we'll learn how to:
- Add user authentication with JWT tokens
- Structure our code better with Flask-RESTful
- Implement proper API versioning
- Add input validation and serialization

## Resources
- [Flask Documentation](https://flask.palletsprojects.com/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [Postman Documentation](https://learning.postman.com/)
- [Flask-CORS Documentation](https://flask-cors.readthedocs.io/)
