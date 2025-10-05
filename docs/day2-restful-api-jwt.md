---
layout: default
title: Day 2 - RESTful API with Flask-RESTful & JWT
nav_order: 3
---

# Day 2: RESTful API with Flask-RESTful & JWT

## Learning Objectives
By the end of this day, you will be able to:
- Understand RESTful API principles
- Create RESTful endpoints using Flask-RESTful
- Implement JWT authentication
- Protect API endpoints with authentication
- Structure code using Flask-RESTful resources
- Handle JSON requests and responses

## Prerequisites
- Completed Day 1 exercises
- Understanding of Flask basics
- Basic knowledge of HTTP methods (GET, POST, PUT, DELETE)

## What is a RESTful API?

### REST Principles
- **Representational State Transfer (REST)**
- Uses HTTP methods to perform operations
- Stateless communication
- JSON as data format
- Resource-based URLs

### HTTP Methods
- `GET` - Retrieve data
- `POST` - Create new data
- `PUT` - Update existing data
- `DELETE` - Remove data

## Setting Up Flask-RESTful

### Installation
```bash
pip install flask-restful flask-jwt-extended
```

### Basic Flask-RESTful Setup
```python
from flask import Flask
from flask_restful import Api, Resource
from flask_jwt_extended import JWTManager, jwt_required, create_access_token

app = Flask(__name__)
api = Api(app)

# JWT Configuration
app.config['JWT_SECRET_KEY'] = 'your-secret-key-here'
jwt = JWTManager(app)
```

## Creating RESTful Resources

### Task Resource Class
```python
from flask_restful import Resource, reqparse
from flask_jwt_extended import jwt_required, get_jwt_identity

class TaskResource(Resource):
    def __init__(self):
        self.parser = reqparse.RequestParser()
        self.parser.add_argument('title', type=str, required=True, help='Title is required')
        self.parser.add_argument('description', type=str, required=False)
        self.parser.add_argument('completed', type=bool, required=False)
    
    @jwt_required()
    def get(self, task_id=None):
        """Get all tasks or a specific task"""
        if task_id:
            task = Task.query.get_or_404(task_id)
            return {
                'id': task.id,
                'title': task.title,
                'description': task.description,
                'completed': task.completed,
                'created_at': task.created_at.isoformat()
            }
        else:
            tasks = Task.query.all()
            return [{
                'id': task.id,
                'title': task.title,
                'description': task.description,
                'completed': task.completed,
                'created_at': task.created_at.isoformat()
            } for task in tasks]
    
    @jwt_required()
    def post(self):
        """Create a new task"""
        args = self.parser.parse_args()
        
        new_task = Task(
            title=args['title'],
            description=args.get('description', ''),
            completed=args.get('completed', False)
        )
        
        db.session.add(new_task)
        db.session.commit()
        
        return {
            'message': 'Task created successfully',
            'task': {
                'id': new_task.id,
                'title': new_task.title,
                'description': new_task.description,
                'completed': new_task.completed
            }
        }, 201
    
    @jwt_required()
    def put(self, task_id):
        """Update an existing task"""
        task = Task.query.get_or_404(task_id)
        args = self.parser.parse_args()
        
        task.title = args['title']
        task.description = args.get('description', task.description)
        task.completed = args.get('completed', task.completed)
        
        db.session.commit()
        
        return {
            'message': 'Task updated successfully',
            'task': {
                'id': task.id,
                'title': task.title,
                'description': task.description,
                'completed': task.completed
            }
        }
    
    @jwt_required()
    def delete(self, task_id):
        """Delete a task"""
        task = Task.query.get_or_404(task_id)
        db.session.delete(task)
        db.session.commit()
        
        return {'message': 'Task deleted successfully'}
```

### Adding Routes
```python
# Add routes to API
api.add_resource(TaskResource, '/api/tasks', '/api/tasks/<int:task_id>')
```

## JWT Authentication

### What is JWT?
- **JSON Web Token** - A secure way to transmit information
- Contains user information and expiration time
- Stateless authentication
- Widely used in modern web applications

### User Model
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128))
    
    def __repr__(self):
        return f'<User {self.username}>'
```

### Authentication Endpoints
```python
from werkzeug.security import generate_password_hash, check_password_hash

class AuthResource(Resource):
    def __init__(self):
        self.parser = reqparse.RequestParser()
        self.parser.add_argument('username', type=str, required=True)
        self.parser.add_argument('password', type=str, required=True)
        self.parser.add_argument('email', type=str, required=False)
    
    def post(self):
        """User registration"""
        args = self.parser.parse_args()
        
        # Check if user already exists
        if User.query.filter_by(username=args['username']).first():
            return {'message': 'Username already exists'}, 400
        
        # Create new user
        user = User(
            username=args['username'],
            email=args.get('email', ''),
            password_hash=generate_password_hash(args['password'])
        )
        
        db.session.add(user)
        db.session.commit()
        
        return {'message': 'User created successfully'}, 201
    
    def put(self):
        """User login"""
        args = self.parser.parse_args()
        
        user = User.query.filter_by(username=args['username']).first()
        
        if user and check_password_hash(user.password_hash, args['password']):
            access_token = create_access_token(identity=user.id)
            return {
                'access_token': access_token,
                'user_id': user.id,
                'username': user.username
            }
        else:
            return {'message': 'Invalid credentials'}, 401
```

### Adding Auth Routes
```python
api.add_resource(AuthResource, '/api/auth/register', '/api/auth/login')
```

## Protecting Endpoints

### Using JWT Required Decorator
```python
@jwt_required()
def get(self):
    current_user_id = get_jwt_identity()
    # Your protected code here
```

### Error Handling
```python
@jwt.expired_token_loader
def expired_token_callback(jwt_header, jwt_payload):
    return {'message': 'Token has expired'}, 401

@jwt.invalid_token_loader
def invalid_token_callback(error):
    return {'message': 'Invalid token'}, 401

@jwt.unauthorized_loader
def missing_token_callback(error):
    return {'message': 'Authorization token is required'}, 401
```

## API Testing

### Using curl (Command Line)
```bash
# Register a new user
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "testuser", "password": "testpass", "email": "test@example.com"}'

# Login
curl -X PUT http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "testuser", "password": "testpass"}'

# Get tasks (with token)
curl -X GET http://localhost:5000/api/tasks \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"

# Create a task
curl -X POST http://localhost:5000/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{"title": "Learn Flask", "description": "Study Flask framework"}'
```

### Using Postman
1. Create a new request
2. Set method to POST/GET/PUT/DELETE
3. Add URL: `http://localhost:5000/api/tasks`
4. Add headers: `Content-Type: application/json`
5. Add Authorization header: `Bearer YOUR_TOKEN`
6. Add JSON body for POST/PUT requests

## Code Organization

### Project Structure
```
2/
├── app.py                 # Main application file
├── models.py             # Database models
├── api/
│   ├── __init__.py
│   ├── tasks.py          # Task API endpoints
│   └── auth.py           # Authentication endpoints
├── requirements.txt
└── database.db
```

### Separating Concerns
```python
# api/tasks.py
from flask_restful import Resource, reqparse
from flask_jwt_extended import jwt_required, get_jwt_identity
from models import Task, db

class TaskResource(Resource):
    # Task-related code here
    pass

# api/auth.py
from flask_restful import Resource, reqparse
from flask_jwt_extended import create_access_token
from models import User, db

class AuthResource(Resource):
    # Authentication code here
    pass
```

## Hands-On Exercises

### Exercise 1: Basic REST API
1. Convert your Day 1 Flask app to use Flask-RESTful
2. Create TaskResource with GET, POST, PUT, DELETE methods
3. Test all endpoints using curl or Postman

### Exercise 2: User Authentication
1. Create User model with username, email, password
2. Implement registration and login endpoints
3. Generate JWT tokens on successful login

### Exercise 3: Protected Endpoints
1. Add `@jwt_required()` decorator to TaskResource methods
2. Test that endpoints require authentication
3. Verify that valid tokens allow access

### Exercise 4: Error Handling
1. Add proper error responses for invalid data
2. Handle cases where resources don't exist
3. Add JWT error handlers

## Key Concepts Summary

### RESTful API Concepts
- **Resources**: Objects that can be accessed via URLs
- **HTTP Methods**: Standard operations (GET, POST, PUT, DELETE)
- **Status Codes**: HTTP response codes (200, 201, 400, 401, 404)
- **JSON**: Data format for requests and responses

### JWT Authentication
- **Token-based**: No server-side session storage
- **Stateless**: Each request contains all necessary information
- **Secure**: Tokens are signed and can be verified
- **Expiration**: Tokens have limited lifetime

### Best Practices
- Use meaningful HTTP status codes
- Validate input data
- Handle errors gracefully
- Keep authentication logic separate
- Use environment variables for secrets

## Common Issues and Solutions

### Issue: JWT token not working
**Solution**: Check that the Authorization header format is correct: `Bearer TOKEN`

### Issue: CORS errors
**Solution**: Install and configure flask-cors for cross-origin requests

### Issue: Database not updating
**Solution**: Make sure to call `db.session.commit()` after making changes


## Resources
- [Flask-RESTful Documentation](https://flask-restful.readthedocs.io/)
- [Flask-JWT-Extended Documentation](https://flask-jwt-extended.readthedocs.io/)
- [REST API Design Guide](https://restfulapi.net/)
- [JWT.io](https://jwt.io/) - JWT token decoder
