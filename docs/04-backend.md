---
layout: default
title: backend boilerplate template 
nav_order: 1
---


# Flask Boilerplate Template Documentation

## 🚀 Quick Start Guide

### Prerequisites
- Python 3.7+
- Redis server
- SMTP credentials (Gmail recommended)

### Installation Steps

1. **Install Dependencies**
```bash
pip install flask flask-restful flask-sqlalchemy flask-jwt-extended flask-cors flask-mail flask-caching celery redis python-dotenv werkzeug
```

2. **Environment Setup**
Create a `.env` file in your project root:
```env
# Email Configuration
MAIL_USER=your_email@gmail.com
MAIL_PASS=your_app_password

# Redis Configuration
BROKER_URL=redis://localhost:6379/0
RESULT_BACKEND=redis://localhost:6379/0

# Security (Change in production!)
JWT_SECRET_KEY=your-super-secret-jwt-key-change-in-production

# Database (Optional - defaults to SQLite)
DATABASE_URL=sqlite:///app.db
```

3. **Start Redis Server**
```bash
# On macOS
brew services start redis

# On Ubuntu
sudo systemctl start redis-server

# On Windows (with Redis installed)
redis-server
```

4. **Run the Application**
```bash
python app.py
```

5. **Start Celery Worker (in another terminal)**
```bash
celery -A app.celery worker --loglevel=info
```

6. **Start Celery Beat for Scheduled Tasks (optional)**
```bash
celery -A app.celery beat --loglevel=info
```

## 📊 Database Models

### User Model
```python
# Fields: id, username, email, password_hash, role, is_active, is_approved, created_at, last_login
# Roles: admin, manager, employee, customer
# Relationships: items (one-to-many), orders (one-to-many)
```

### Category Model
```python
# Fields: id, name, description, is_active, created_at
# Relationships: items (one-to-many)
```

### Item Model (Flexible Core Entity)
```python
# Fields: id, title, description, price, status, priority, tags (JSON), metadata (JSON)
# Fields: category_id, owner_id, created_at, updated_at, due_date
# Use Cases:
# - E-commerce: Product catalog
# - Task Management: Tasks/Issues
# - Quiz System: Questions
# - Parking: Parking spots
# - Event Management: Events
```

### Order Model
```python
# Fields: id, order_number, total_amount, status, notes, customer_id, created_at, updated_at
# Relationships: items (many-to-many through OrderItem), customer (many-to-one)
```

### OrderItem Model (Junction Table)
```python
# Fields: id, quantity, unit_price, total_price, order_id, item_id
```

## 🔐 Authentication & Authorization

### Registration Flow
```bash
POST /api/auth/register
{
    "username": "newuser",
    "email": "user@example.com",
    "password": "securepassword",
    "role": "customer"  # optional, defaults to customer
}
```

### Login Flow
```bash
POST /api/auth/login
{
    "email": "user@example.com",
    "password": "securepassword"
}
# Returns JWT token and user info
```

### Role-Based Access Control
- **Admin**: Full system access, user management
- **Manager**: Content management, user oversight
- **Employee**: Limited content access, own items
- **Customer**: Basic access, own orders

### Using JWT Token
```bash
# Include in all authenticated requests
Authorization: Bearer <your-jwt-token>
```

## 📚 API Endpoints Reference

### Authentication Endpoints
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/auth/register` | User registration | No |
| POST | `/api/auth/login` | User login | No |
| POST | `/api/auth/logout` | User logout | Yes |
| GET | `/api/auth/profile` | Get user profile | Yes |
| PUT | `/api/auth/profile` | Update user profile | Yes |

### User Management (Admin Only)
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/users` | Get all users | Admin |
| PUT | `/api/users/<id>` | Update user | Admin |

### Category Management
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/categories` | Get categories | No |
| POST | `/api/categories` | Create category | Admin/Manager |
| PUT | `/api/categories/<id>` | Update category | Admin/Manager |

### Item Management
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/items` | Get items (cached) | No |
| GET | `/api/items/<id>` | Get specific item | No |
| POST | `/api/items` | Create item | Yes |
| PUT | `/api/items/<id>` | Update item | Yes (Owner/Admin/Manager) |
| DELETE | `/api/items/<id>` | Delete item (soft) | Admin/Manager |

### Order Management
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/orders` | Get orders | Yes |
| GET | `/api/orders/<id>` | Get specific order | Yes (Owner/Admin/Manager) |
| POST | `/api/orders` | Create order | Yes |
| PUT | `/api/orders/<id>` | Update order | Admin/Manager |

### File Management
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/upload` | Upload file | Yes |
| GET | `/api/files/<filename>` | Get uploaded file | No |

### Background Tasks & Email
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/send-email` | Send email | Admin/Manager |
| GET | `/api/tasks/<task_id>` | Get task status | Yes |

### Analytics & Caching
| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | `/api/analytics` | Get system analytics | Admin/Manager |
| GET | `/api/cache-demo` | Cache demonstration | No |
| DELETE | `/api/cache-demo` | Clear demo cache | No |
| GET | `/api/health` | Health check | No |

## 🎯 Project Adaptation Examples

### 1. Quiz Management System
```python
# Adapt models:
# Item = Question
# Category = Subject/Topic
# Order = Quiz Attempt
# OrderItem = Answer

# Additional fields in Item.metadata:
{
    "question_type": "multiple_choice",
    "options": ["A", "B", "C", "D"],
    "correct_answer": "B",
    "points": 10,
    "difficulty": "medium"
}

# Additional fields in Order.metadata:
{
    "quiz_name": "Python Basics",
    "time_limit": 60,
    "score": 85,
    "passed": true
}
```

### 2. Parking Management System
```python
# Adapt models:
# Item = Parking Spot
# Category = Zone/Floor
# Order = Booking
# OrderItem = Booking Details

# Additional fields in Item.metadata:
{
    "spot_number": "A-101",
    "spot_type": "standard",
    "is_covered": true,
    "is_electric_charging": false,
    "floor": 1
}

# Additional fields in Order.metadata:
{
    "vehicle_number": "ABC-1234",
    "check_in": "2024-01-15T09:00:00Z",
    "check_out": "2024-01-15T17:00:00Z",
    "total_hours": 8
}
```

### 3. Event Management System
```python
# Adapt models:
# Item = Event
# Category = Event Type
# Order = Registration
# OrderItem = Ticket

# Additional fields in Item.metadata:
{
    "event_date": "2024-02-15T18:00:00Z",
    "location": "Convention Center",
    "max_attendees": 500,
    "current_attendees": 245,
    "ticket_types": ["VIP", "Standard", "Student"]
}
```

### 4. Task Management System
```python
# Adapt models:
# Item = Task
# Category = Project
# Order = Sprint
# OrderItem = Sprint Task

# Additional fields in Item.metadata:
{
    "estimated_hours": 8,
    "actual_hours": 10,
    "assignee": "john_doe",
    "labels": ["bug", "urgent"],
    "sprint_id": 5
}
```

## ⚡ Caching Strategy

### Cache Decorators
```python
# Method 1: Cached function
@cache.cached(timeout=300)  # 5 minutes
def expensive_function():
    pass

# Method 2: Memoized function (with arguments)
@cache.memoize(timeout=300)
def function_with_args(arg1, arg2):
    pass

# Method 3: Manual caching
def manual_cache_example():
    key = "custom_cache_key"
    data = cache.get(key)
    if data is None:
        data = expensive_computation()
        cache.set(key, data, timeout=300)
    return data
```

### Cache Management
```python
# Clear specific cache
cache.delete_memoized(function_name)

# Clear all cache
cache.clear()

# Cache with Redis directly
redis_client.set("key", "value", ex=300)  # 5 minutes
value = redis_client.get("key")
```

## 🔄 Background Tasks (Celery)

### Defining Tasks
```python
@celery.task(name='app.custom_task')
def custom_background_task(param1, param2):
    # Your background logic here
    return result

# Usage
task = custom_background_task.delay(arg1, arg2)
task_id = task.id
```

### Scheduled Tasks (Celery Beat)
```python
# Add to celery.conf.beat_schedule
celery.conf.beat_schedule = {
    'daily-cleanup': {
        'task': 'app.cleanup_task',
        'schedule': crontab(hour=2, minute=0),  # Daily at 2 AM
    },
    'weekly-report': {
        'task': 'app.generate_weekly_report',
        'schedule': crontab(day_of_week=1, hour=9, minute=0),  # Monday 9 AM
    }
}
```

### Task Monitoring
```python
# Check task status
GET /api/tasks/<task_id>

# Response
{
    "task_id": "abc-123",
    "status": "SUCCESS",
    "result": "Task completed successfully",
    "info": {}
}
```

## 📧 Email System

### Email Templates
```python
# Send simple email
send_email_task.delay(
    to_email="user@example.com",
    subject="Welcome!",
    body="Welcome to our platform!"
)

# Send HTML email (extend the task)
@celery.task(name='app.send_html_email')
def send_html_email(to_email, subject, html_body):
    msg = Message(
        subject=subject,
        recipients=[to_email],
        html=html_body
    )
    mail.send(msg)
```

### Email Notifications
```python
# User registration
send_email_task.delay(
    user.email,
    "Welcome to our platform!",
    f"Hello {user.username}, welcome!"
)

# Order confirmation
send_email_task.delay(
    order.customer.email,
    f"Order Confirmation - {order.order_number}",
    f"Your order has been confirmed!"
)
```

## 🛠️ CLI Commands

### Available Commands
```bash
# Initialize database with admin user
flask init-db

# Clear all cache
flask clear-cache

# Create sample data for testing
flask create-sample-data
```

### Custom CLI Commands
```python
@app.cli.command()
def your_custom_command():
    """Your custom command description"""
    # Your logic here
    print("Command executed!")

# Usage: flask your-custom-command
```

## 🔧 Configuration Management

### Environment-Specific Configs
```python
# Development
app.config['DEBUG'] = True
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///dev.db'

# Testing
app.config['TESTING'] = True
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'

# Production
app.config['DEBUG'] = False
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL')
```

### Security Best Practices
```python
# Use strong JWT secret key
JWT_SECRET_KEY = os.urandom(32).hex()

# Hash all passwords
user.set_password(plain_password)

# Validate file uploads
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'}

# Limit file size
MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16MB

# Enable CORS selectively
CORS(app, resources={r"/api/*": {"origins": ["https://yourdomain.com"]}})
```

## 📊 Database Relationships & Queries

### Common Query Patterns
```python
# Get items with category and owner info
items = db.session.query(Item)\
    .join(Category)\
    .join(User)\
    .filter(Item.status == 'active')\
    .all()

# Get user's orders with items
orders = db.session.query(Order)\
    .join(OrderItem)\
    .join(Item)\
    .filter(Order.customer_id == user_id)\
    .all()

# Analytics queries
revenue_by_month = db.session.query(
    db.func.date_trunc('month', Order.created_at),
    db.func.sum(Order.total_amount)
).group_by(db.func.date_trunc('month', Order.created_at)).all()

# Items by category count
category_stats = db.session.query(
    Category.name,
    db.func.count(Item.id)
).join(Item).group_by(Category.name).all()
```

### Database Migrations (Flask-Migrate)
```bash
# Install Flask-Migrate
pip install Flask-Migrate

# Initialize migrations
flask db init

# Create migration
flask db migrate -m "Add new field"

# Apply migration
flask db upgrade
```

## 🚀 Production Deployment

### Docker Configuration
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - BROKER_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6-alpine

  celery:
    build: .
    command: celery -A app.celery worker --loglevel=info
    depends_on:
      - redis
      - db

  celery-beat:
    build: .
    command: celery -A app.celery beat --loglevel=info
    depends_on:
      - redis
      - db

volumes:
  postgres_data:
```

### Production Environment Variables
```env
# Production .env
DEBUG=False
SECRET_KEY=your-super-secret-production-key
DATABASE_URL=postgresql://user:password@localhost/production_db
BROKER_URL=redis://localhost:6379/0
RESULT_BACKEND=redis://localhost:6379/0

# Email (Production SMTP)
MAIL_SERVER=smtp.sendgrid.net
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=apikey
MAIL_PASSWORD=your-sendgrid-api-key

# Security
JWT_SECRET_KEY=your-production-jwt-secret
CORS_ORIGINS=https://yourapp.com,https://api.yourapp.com
```

## 🧪 Testing

### Test Configuration
```python
# test_config.py
import unittest
from app import create_app, db

class TestConfig:
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    JWT_SECRET_KEY = 'test-secret-key'
    WTF_CSRF_ENABLED = False

class BaseTestCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app('testing')
        self.app_context = self.app.app_context()
        self.app_context.push()
        self.client = self.app.test_client()
        db.create_all()

    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()
```

### Example Tests
```python
# test_auth.py
class AuthTestCase(BaseTestCase):
    def test_user_registration(self):
        response = self.client.post('/api/auth/register', json={
            'username': 'testuser',
            'email': 'test@example.com',
            'password': 'testpass123'
        })
        self.assertEqual(response.status_code, 201)
        data = response.get_json()
        self.assertIn('user', data)

    def test_user_login(self):
        # Create user first
        user = User(username='testuser', email='test@example.com')
        user.set_password('testpass123')
        db.session.add(user)
        db.session.commit()

        response = self.client.post('/api/auth/login', json={
            'email': 'test@example.com',
            'password': 'testpass123'
        })
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
        self.assertIn('access_token', data)
```

### Running Tests
```bash
# Install testing dependencies
pip install pytest pytest-cov

# Run tests
python -m pytest

# Run with coverage
python -m pytest --cov=app tests/
```

## 📈 Performance Optimization

### Database Optimization
```python
# Use indexes
class User(db.Model):
    username = db.Column(db.String(80), unique=True, index=True)
    email = db.Column(db.String(120), unique=True, index=True)

# Query optimization
# Bad: N+1 queries
for item in items:
    print(item.category.name)

# Good: Join query
items = Item.query.options(db.joinedload(Item.category)).all()
for item in items:
    print(item.category.name)

# Pagination for large datasets
items = Item.query.paginate(page=1, per_page=20, error_out=False)
```

### Caching Strategies
```python
# Cache expensive queries
@cache.memoize(timeout=3600)  # 1 hour
def get_popular_items():
    return Item.query.filter(Item.status == 'active')\
        .order_by(Item.view_count.desc())\
        .limit(10).all()

# Cache user sessions
@cache.memoize(timeout=1800)  # 30 minutes
def get_user_permissions(user_id):
    user = User.query.get(user_id)
    return user.role if user else None
```

### Background Task Optimization
```python
# Use batch processing
@celery.task
def process_bulk_emails(email_list):
    with mail.connect() as conn:
        for email_data in email_list:
            msg = Message(**email_data)
            conn.send(msg)

# Use result grouping
from celery import group
job = group(send_email_task.s(email) for email in email_list)
result = job.apply_async()
```

## 🔐 Security Checklist

### Authentication Security
- [x] Password hashing with bcrypt
- [x] JWT token expiration
- [x] Token blacklisting on logout
- [x] Role-based access control
- [x] Input validation and sanitization

### API Security
```python
# Rate limiting (add Flask-Limiter)
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/api/auth/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    pass

# CORS configuration
CORS(app, resources={
    r"/api/*": {
        "origins": ["https://yourdomain.com"],
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "allow_headers": ["Content-Type", "Authorization"]
    }
})

# Request validation
from flask_marshmallow import Marshmallow
ma = Marshmallow(app)

class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User
        load_instance = True
```

## 📚 Advanced Features

### WebSocket Support (Flask-SocketIO)
```python
from flask_socketio import SocketIO, emit

socketio = SocketIO(app, cors_allowed_origins="*")

@socketio.on('connect')
@jwt_required()
def handle_connect():
    user = get_current_user()
    emit('status', {'msg': f'{user.username} connected'})

@socketio.on('order_update')
def handle_order_update(data):
    # Broadcast order updates to relevant users
    emit('order_status', data, broadcast=True)
```

### API Versioning
```python
# URL versioning
api_v1 = Api(app, prefix='/api/v1')
api_v2 = Api(app, prefix='/api/v2')

# Header versioning
@app.before_request
def before_request():
    version = request.headers.get('API-Version', 'v1')
    g.api_version = version
```

### File Processing with Background Tasks
```python
@celery.task
def process_uploaded_file(file_path):
    # Process CSV, images, documents, etc.
    if file_path.endswith('.csv'):
        import pandas as pd
        df = pd.read_csv(file_path)
        # Process data
        return f"Processed {len(df)} rows"
    
    elif file_path.endswith('.jpg'):
        from PIL import Image
        img = Image.open(file_path)
        # Resize, optimize, etc.
        return "Image processed"
```

### Real-time Notifications
```python
@celery.task
def send_notification(user_id, message, notification_type='info'):
    # Store in database
    notification = Notification(
        user_id=user_id,
        message=message,
        type=notification_type
    )
    db.session.add(notification)
    db.session.commit()
    
    # Send via WebSocket
    socketio.emit('notification', {
        'message': message,
        'type': notification_type
    }, room=f'user_{user_id}')
    
    # Send via email if critical
    if notification_type == 'critical':
        user = User.query.get(user_id)
        send_email_task.delay(user.email, 'Critical Alert', message)
```

## 🐛 Debugging & Monitoring

### Logging Configuration
```python
import logging
from logging.handlers import RotatingFileHandler

if not app.debug:
    file_handler = RotatingFileHandler('logs/app.log', maxBytes=10240, backupCount=10)
    file_handler.setFormatter(logging.Formatter(
        '%(asctime)s %(levelname)s: %(message)s [in %(pathname)s:%(lineno)d]'
    ))
    file_handler.setLevel(logging.INFO)
    app.logger.addHandler(file_handler)
    app.logger.setLevel(logging.INFO)
```

### Health Check Endpoint
```python
@app.route('/api/health/detailed')
def detailed_health_check():
    health_status = {
        'status': 'healthy',
        'timestamp': datetime.utcnow().isoformat(),
        'checks': {}
    }
    
    # Database check
    try:
        db.session.execute('SELECT 1')
        health_status['checks']['database'] = 'healthy'
    except Exception as e:
        health_status['checks']['database'] = f'unhealthy: {str(e)}'
        health_status['status'] = 'unhealthy'
    
    # Redis check
    try:
        redis_client.ping()
        health_status['checks']['redis'] = 'healthy'
    except Exception as e:
        health_status['checks']['redis'] = f'unhealthy: {str(e)}'
        health_status['status'] = 'unhealthy'
    
    # Celery check
    try:
        i = celery.control.inspect()
        stats = i.stats()
        health_status['checks']['celery'] = 'healthy' if stats else 'no workers'
    except Exception as e:
        health_status['checks']['celery'] = f'unhealthy: {str(e)}'
    
    return jsonify(health_status)
```

## 🔄 Continuous Integration/Deployment

### GitHub Actions Workflow
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      redis:
        image: redis:6
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest pytest-cov
    
    - name: Run tests
      run: |
        pytest --cov=app tests/
    
    - name: Upload coverage
      uses: codecov/codecov-action@v1
```

## 📝 Customization Examples

### Quiz Management System Implementation
```python
# models.py additions
class Quiz(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    description = db.Column(db.Text)
    time_limit = db.Column(db.Integer)  # minutes
    passing_score = db.Column(db.Float)  # percentage
    category_id = db.Column(db.Integer, db.ForeignKey('categories.id'))
    created_by = db.Column(db.Integer, db.ForeignKey('users.id'))
    is_active = db.Column(db.Boolean, default=True)
    
    questions = db.relationship('Question', backref='quiz', lazy=True)
    attempts = db.relationship('QuizAttempt', backref='quiz', lazy=True)

class Question(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    quiz_id = db.Column(db.Integer, db.ForeignKey('quiz.id'), nullable=False)
    question_text = db.Column(db.Text, nullable=False)
    question_type = db.Column(db.String(50), default='multiple_choice')
    options = db.Column(db.Text)  # JSON array
    correct_answer = db.Column(db.String(500))
    points = db.Column(db.Integer, default=1)
    explanation = db.Column(db.Text)

class QuizAttempt(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    quiz_id = db.Column(db.Integer, db.ForeignKey('quiz.id'), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    score = db.Column(db.Float)
    total_points = db.Column(db.Integer)
    time_taken = db.Column(db.Integer)  # seconds
    completed_at = db.Column(db.DateTime)
    answers = db.Column(db.Text)  # JSON object
```

## 📞 Support & Resources

### Common Issues & Solutions

**Issue: Celery worker not starting**
```bash
# Solution: Check Redis connection
redis-cli ping

# Check Celery configuration
celery -A app.celery inspect stats
```

**Issue: JWT token expired**
```python
# Solution: Implement token refresh
@app.route('/api/auth/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh():
    current_user = get_jwt_identity()
    new_token = create_access_token(identity=current_user)
    return {'access_token': new_token}
```

**Issue: CORS errors**
```python
# Solution: Configure CORS properly
CORS(app, resources={
    r"/api/*": {
        "origins": ["http://localhost:3000", "https://yourdomain.com"],
        "methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
        "allow_headers": ["Content-Type", "Authorization"]
    }
})
```

### Useful Extensions
- **Flask-Admin**: Admin interface
- **Flask-Migrate**: Database migrations
- **Flask-Limiter**: Rate limiting
- **Flask-SocketIO**: WebSocket support
- **Flask-Marshmallow**: Serialization
- **Flask-Babel**: Internationalization
- **Celery-Beat**: Scheduled tasks
- **APScheduler**: Alternative task scheduler

### Community Resources
- [Flask Documentation](https://flask.palletsprojects.com/)
- [Flask-RESTful Guide](https://flask-restful.readthedocs.io/)
- [Celery Documentation](https://docs.celeryproject.org/)
- [SQLAlchemy ORM](https://docs.sqlalchemy.org/)
- [JWT Best Practices](https://auth0.com/blog/a-look-at-the-latest-draft-for-jwt-bcp/)

### Contributing Guidelines
1. Fork the repository
2. Create a feature branch
3. Write tests for new features
4. Ensure all tests pass
5. Submit a pull request with clear description

---
