 ```python


"""
Flask Boilerplate Template
==========================

A comprehensive Flask application template with REST APIs, background jobs, caching, 
user authentication, and database models. Perfect for projects like quiz systems, 
parking management, task management, e-commerce, etc.

Features:
- JWT Authentication & Role-based Access Control
- RESTful APIs with Flask-RESTful
- Background Jobs with Celery
- Caching with Redis
- Email notifications
- File upload handling
- SQLAlchemy ORM with example models
- CORS support
- Comprehensive error handling

Prerequisites:
- Python 3.7+
- Redis server (for caching and Celery broker)
- SMTP credentials (for email functionality)

Installation:
1. pip install flask flask-restful flask-sqlalchemy flask-jwt-extended flask-cors flask-mail flask-caching celery redis python-dotenv werkzeug
2. Create .env file with your configurations
3. Start Redis server
4. Run: python app.py

Environment Variables (.env):
MAIL_USER=your_email@gmail.com
MAIL_PASS=your_app_password
BROKER_URL=redis://localhost:6379/0
RESULT_BACKEND=redis://localhost:6379/0
"""

import os
import json
from datetime import datetime, timedelta
from functools import wraps

# Flask imports
from flask import Flask, request, jsonify, send_from_directory
from flask_restful import Api, Resource
from flask_sqlalchemy import SQLAlchemy
from flask_jwt_extended import (
    JWTManager, create_access_token, get_jwt_identity, 
    jwt_required, get_jwt
)
from flask_cors import CORS
from flask_mail import Mail, Message
from flask_caching import Cache

# Security and utilities
from werkzeug.security import generate_password_hash, check_password_hash
from werkzeug.utils import secure_filename

# Background jobs
from celery import Celery
from celery.schedules import crontab

# Environment and Redis
from dotenv import load_dotenv
import redis

# Load environment variables
load_dotenv()

# ==========================
# Flask Application Setup
# ==========================

app = Flask(__name__)

# Database Configuration
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL', 'sqlite:///app.db')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# JWT Configuration
app.config['JWT_SECRET_KEY'] = os.getenv('JWT_SECRET_KEY', 'your-super-secret-jwt-key-change-in-production')
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(hours=24)

# File Upload Configuration
UPLOAD_FOLDER = os.path.abspath(os.path.join(os.path.dirname(__file__), 'uploads'))
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif', 'doc', 'docx'}
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16MB max file size

# Email Configuration
app.config['MAIL_SERVER'] = 'smtp.gmail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.getenv('MAIL_USER')
app.config['MAIL_PASSWORD'] = os.getenv('MAIL_PASS')
app.config['MAIL_DEFAULT_SENDER'] = os.getenv('MAIL_USER')

# Celery Configuration
app.config['CELERY_BROKER_URL'] = os.getenv('BROKER_URL', 'redis://localhost:6379/0')
app.config['CELERY_RESULT_BACKEND'] = os.getenv('RESULT_BACKEND', 'redis://localhost:6379/0')

# Caching Configuration
app.config['CACHE_TYPE'] = 'redis'
app.config['CACHE_REDIS_URL'] = os.getenv('BROKER_URL', 'redis://localhost:6379/0')

# Create upload directory if it doesn't exist
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

# Initialize extensions
db = SQLAlchemy(app)
jwt = JWTManager(app)
api = Api(app)
mail = Mail(app)
cache = Cache(app)
redis_client = redis.Redis.from_url(app.config['CACHE_REDIS_URL'], decode_responses=True)

# Enable CORS
CORS(app, resources={r"/api/*": {"origins": "*"}})

# ==========================
# Celery Setup
# ==========================

def make_celery(app):
    """Create and configure Celery instance"""
    celery = Celery(
        app.import_name,
        backend=app.config['CELERY_RESULT_BACKEND'],
        broker=app.config['CELERY_BROKER_URL']
    )
    celery.conf.update(app.config)
    celery.conf.broker_connection_retry_on_startup = True

    class ContextTask(celery.Task):
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return super().__call__(*args, **kwargs)

    celery.Task = ContextTask
    return celery

celery = make_celery(app)

# ==========================
# Database Models
# ==========================

class User(db.Model):
    """
    User model for authentication and role management
    
    Roles: admin, manager, employee, customer
    """
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False, index=True)
    email = db.Column(db.String(120), unique=True, nullable=False, index=True)
    password_hash = db.Column(db.String(128), nullable=False)
    role = db.Column(db.String(20), nullable=False, default='customer')
    is_active = db.Column(db.Boolean, default=True)
    is_approved = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_login = db.Column(db.DateTime)
    
    # Relationships
    items = db.relationship('Item', backref='owner', lazy=True)
    orders = db.relationship('Order', backref='customer', lazy=True)
    
    def set_password(self, password):
        """Hash and set password"""
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        """Check if provided password matches hash"""
        return check_password_hash(self.password_hash, password)
    
    def to_dict(self):
        """Convert user to dictionary"""
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'role': self.role,
            'is_active': self.is_active,
            'is_approved': self.is_approved,
            'created_at': self.created_at.isoformat() if self.created_at else None,
            'last_login': self.last_login.isoformat() if self.last_login else None
        }

class Category(db.Model):
    """Category model for organizing items"""
    __tablename__ = 'categories'
    
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False, unique=True)
    description = db.Column(db.Text)
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    items = db.relationship('Item', backref='category', lazy=True)
    
    def to_dict(self):
        return {
            'id': self.id,
            'name': self.name,
            'description': self.description,
            'is_active': self.is_active,
            'created_at': self.created_at.isoformat() if self.created_at else None,
            'items_count': len(self.items)
        }

class Item(db.Model):
    """
    Generic Item model - can represent products, tasks, quizzes, parking spots, etc.
    Adapt fields based on your use case
    """
    __tablename__ = 'items'
    
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    description = db.Column(db.Text)
    price = db.Column(db.Float, default=0.0)  # For e-commerce
    status = db.Column(db.String(50), default='active')  # active, inactive, completed, etc.
    priority = db.Column(db.String(20), default='medium')  # low, medium, high
    tags = db.Column(db.Text)  # JSON string for flexible tagging
    metadata = db.Column(db.Text)  # JSON string for additional data
    
    # Foreign Keys
    category_id = db.Column(db.Integer, db.ForeignKey('categories.id'))
    owner_id = db.Column(db.Integer, db.ForeignKey('users.id'))
    
    # Timestamps
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    due_date = db.Column(db.DateTime)  # For tasks/deadlines
    
    # Relationships
    order_items = db.relationship('OrderItem', backref='item', lazy=True)
    
    def get_tags(self):
        """Parse tags from JSON string"""
        try:
            return json.loads(self.tags) if self.tags else []
        except:
            return []
    
    def set_tags(self, tags_list):
        """Set tags as JSON string"""
        self.tags = json.dumps(tags_list) if tags_list else None
    
    def get_metadata(self):
        """Parse metadata from JSON string"""
        try:
            return json.loads(self.metadata) if self.metadata else {}
        except:
            return {}
    
    def set_metadata(self, meta_dict):
        """Set metadata as JSON string"""
        self.metadata = json.dumps(meta_dict) if meta_dict else None
    
    def to_dict(self):
        return {
            'id': self.id,
            'title': self.title,
            'description': self.description,
            'price': self.price,
            'status': self.status,
            'priority': self.priority,
            'tags': self.get_tags(),
            'metadata': self.get_metadata(),
            'category_id': self.category_id,
            'owner_id': self.owner_id,
            'created_at': self.created_at.isoformat() if self.created_at else None,
            'updated_at': self.updated_at.isoformat() if self.updated_at else None,
            'due_date': self.due_date.isoformat() if self.due_date else None,
            'category': self.category.name if self.category else None,
            'owner': self.owner.username if self.owner else None
        }

class Order(db.Model):
    """Order model for e-commerce or booking systems"""
    __tablename__ = 'orders'
    
    id = db.Column(db.Integer, primary_key=True)
    order_number = db.Column(db.String(50), unique=True, nullable=False)
    total_amount = db.Column(db.Float, default=0.0)
    status = db.Column(db.String(50), default='pending')  # pending, confirmed, completed, cancelled
    notes = db.Column(db.Text)
    
    # Foreign Keys
    customer_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    
    # Timestamps
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # Relationships
    items = db.relationship('OrderItem', backref='order', lazy=True, cascade='all, delete-orphan')
    
    def to_dict(self):
        return {
            'id': self.id,
            'order_number': self.order_number,
            'total_amount': self.total_amount,
            'status': self.status,
            'notes': self.notes,
            'customer_id': self.customer_id,
            'customer_name': self.customer.username if self.customer else None,
            'created_at': self.created_at.isoformat() if self.created_at else None,
            'updated_at': self.updated_at.isoformat() if self.updated_at else None,
            'items': [item.to_dict() for item in self.items]
        }

class OrderItem(db.Model):
    """Junction table for orders and items"""
    __tablename__ = 'order_items'
    
    id = db.Column(db.Integer, primary_key=True)
    quantity = db.Column(db.Integer, default=1)
    unit_price = db.Column(db.Float, nullable=False)
    total_price = db.Column(db.Float, nullable=False)
    
    # Foreign Keys
    order_id = db.Column(db.Integer, db.ForeignKey('orders.id'), nullable=False)
    item_id = db.Column(db.Integer, db.ForeignKey('items.id'), nullable=False)
    
    def to_dict(self):
        return {
            'id': self.id,
            'quantity': self.quantity,
            'unit_price': self.unit_price,
            'total_price': self.total_price,
            'order_id': self.order_id,
            'item_id': self.item_id,
            'item_title': self.item.title if self.item else None
        }

# ==========================
# JWT and Authentication Helpers
# ==========================

# Store blacklisted tokens (in production, use Redis or database)
blacklisted_tokens = set()

@jwt.token_in_blocklist_loader
def check_if_token_revoked(jwt_header, jwt_payload):
    """Check if JWT token is blacklisted"""
    return jwt_payload['jti'] in blacklisted_tokens

def role_required(allowed_roles):
    """Decorator to check user roles"""
    def decorator(f):
        @wraps(f)
        @jwt_required()
        def decorated_function(*args, **kwargs):
            current_user_id = get_jwt_identity()
            user = User.query.get(current_user_id)
            
            if not user or not user.is_active:
                return {'message': 'User not found or inactive'}, 401
            
            if user.role not in allowed_roles:
                return {'message': 'Insufficient permissions'}, 403
                
            return f(*args, **kwargs)
        return decorated_function
    return decorator

def get_current_user():
    """Get current authenticated user"""
    user_id = get_jwt_identity()
    return User.query.get(user_id) if user_id else None

# ==========================
# Utility Functions
# ==========================

def allowed_file(filename):
    """Check if file extension is allowed"""
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

def generate_order_number():
    """Generate unique order number"""
    import uuid
    return f"ORD-{uuid.uuid4().hex[:8].upper()}"

# ==========================
# Background Tasks (Celery)
# ==========================

@celery.task(name='app.send_email_task')
def send_email_task(to_email, subject, body):
    """Background task to send emails"""
    try:
        msg = Message(
            subject=subject,
            recipients=[to_email],
            body=body
        )
        mail.send(msg)
        return f"Email sent successfully to {to_email}"
    except Exception as e:
        return f"Failed to send email: {str(e)}"

@celery.task(name='app.process_order_task')
def process_order_task(order_id):
    """Background task to process orders"""
    with app.app_context():
        order = Order.query.get(order_id)
        if order:
            # Simulate order processing
            import time
            time.sleep(2)  # Simulate processing time
            
            order.status = 'confirmed'
            db.session.commit()
            
            # Send confirmation email
            send_email_task.delay(
                order.customer.email,
                f"Order Confirmation - {order.order_number}",
                f"Your order {order.order_number} has been confirmed!"
            )
            
            return f"Order {order.order_number} processed successfully"
        return "Order not found"

@celery.task(name='app.cleanup_expired_data')
def cleanup_expired_data():
    """Background task to cleanup old data"""
    with app.app_context():
        # Example: Delete items older than 30 days with status 'deleted'
        cutoff_date = datetime.utcnow() - timedelta(days=30)
        expired_items = Item.query.filter(
            Item.status == 'deleted',
            Item.updated_at < cutoff_date
        ).all()
        
        count = len(expired_items)
        for item in expired_items:
            db.session.delete(item)
        
        db.session.commit()
        return f"Cleaned up {count} expired items"

# Configure Celery Beat for scheduled tasks
celery.conf.beat_schedule = {
    'cleanup-expired-data': {
        'task': 'app.cleanup_expired_data',
        'schedule': crontab(hour=2, minute=0),  # Run daily at 2 AM
    },
}
celery.conf.timezone = 'UTC'

# ==========================
# API Resources
# ==========================

class HealthCheck(Resource):
    """Health check endpoint"""
    
    def get(self):
        """Check if application is running"""
        return {
            'status': 'healthy',
            'timestamp': datetime.utcnow().isoformat(),
            'version': '1.0.0'
        }

class AuthRegister(Resource):
    """User registration endpoint"""
    
    def post(self):
        """Register a new user"""
        try:
            data = request.get_json()
            
            # Validate required fields
            required_fields = ['username', 'email', 'password']
            for field in required_fields:
                if field not in data:
                    return {'message': f'{field} is required'}, 400
            
            # Check if user already exists
            if User.query.filter_by(username=data['username']).first():
                return {'message': 'Username already exists'}, 409
            
            if User.query.filter_by(email=data['email']).first():
                return {'message': 'Email already registered'}, 409
            
            # Create new user
            user = User(
                username=data['username'],
                email=data['email'],
                role=data.get('role', 'customer')
            )
            user.set_password(data['password'])
            
            db.session.add(user)
            db.session.commit()
            
            # Send welcome email (background task)
            send_email_task.delay(
                user.email,
                "Welcome to our platform!",
                f"Hello {user.username}, welcome to our platform!"
            )
            
            return {
                'message': 'User registered successfully',
                'user': user.to_dict()
            }, 201
            
        except Exception as e:
            db.session.rollback()
            return {'message': f'Deletion failed: {str(e)}'}, 500

class OrderAPI(Resource):
    """Order management API"""
    
    @jwt_required()
    def get(self, order_id=None):
        """Get orders"""
        user = get_current_user()
        
        if order_id:
            order = Order.query.get_or_404(order_id)
            
            # Check permissions
            if user.role not in ['admin', 'manager'] and order.customer_id != user.id:
                return {'message': 'Permission denied'}, 403
            
            return {'order': order.to_dict()}
        
        # Get user's orders or all orders for admin/manager
        if user.role in ['admin', 'manager']:
            orders = Order.query.all()
        else:
            orders = Order.query.filter_by(customer_id=user.id).all()
        
        return {'orders': [order.to_dict() for order in orders]}
    
    @jwt_required()
    def post(self):
        """Create new order"""
        try:
            data = request.get_json()
            user = get_current_user()
            
            if not data.get('items'):
                return {'message': 'Order items are required'}, 400
            
            # Create order
            order = Order(
                order_number=generate_order_number(),
                customer_id=user.id,
                notes=data.get('notes', '')
            )
            
            db.session.add(order)
            db.session.flush()  # Get order ID
            
            total_amount = 0
            
            # Add order items
            for item_data in data['items']:
                item = Item.query.get(item_data['item_id'])
                if not item:
                    return {'message': f'Item {item_data["item_id"]} not found'}, 404
                
                quantity = item_data.get('quantity', 1)
                unit_price = item.price
                total_price = unit_price * quantity
                
                order_item = OrderItem(
                    order_id=order.id,
                    item_id=item.id,
                    quantity=quantity,
                    unit_price=unit_price,
                    total_price=total_price
                )
                
                db.session.add(order_item)
                total_amount += total_price
            
            order.total_amount = total_amount
            db.session.commit()
            
            # Process order in background
            process_order_task.delay(order.id)
            
            return {'message': 'Order created successfully', 'order': order.to_dict()}, 201
            
        except Exception as e:
            db.session.rollback()
            return {'message': f'Order creation failed: {str(e)}'}, 500
    
    @jwt_required()
    def put(self, order_id):
        """Update order status"""
        try:
            order = Order.query.get_or_404(order_id)
            user = get_current_user()
            
            # Check permissions
            if user.role not in ['admin', 'manager']:
                return {'message': 'Permission denied'}, 403
            
            data = request.get_json()
            
            if 'status' in data:
                order.status = data['status']
            if 'notes' in data:
                order.notes = data['notes']
            
            db.session.commit()
            
            return {'message': 'Order updated successfully', 'order': order.to_dict()}
            
        except Exception as e:
            db.session.rollback()
            return {'message': f'Update failed: {str(e)}'}, 500

class FileUpload(Resource):
    """File upload endpoint"""
    
    @jwt_required()
    def post(self):
        """Upload file"""
        try:
            if 'file' not in request.files:
                return {'message': 'No file provided'}, 400
            
            file = request.files['file']
            
            if file.filename == '':
                return {'message': 'No file selected'}, 400
            
            if file and allowed_file(file.filename):
                filename = secure_filename(file.filename)
                
                # Add timestamp to prevent conflicts
                timestamp = datetime.now().strftime('%Y%m%d_%H%M%S_')
                filename = timestamp + filename
                
                file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
                file.save(file_path)
                
                return {
                    'message': 'File uploaded successfully',
                    'filename': filename,
                    'url': f'/api/files/{filename}'
                }, 201
            
            return {'message': 'File type not allowed'}, 400
            
        except Exception as e:
            return {'message': f'Upload failed: {str(e)}'}, 500
    
    def get(self, filename):
        """Serve uploaded file"""
        try:
            return send_from_directory(app.config['UPLOAD_FOLDER'], filename)
        except FileNotFoundError:
            return {'message': 'File not found'}, 404

class CacheDemo(Resource):
    """Cache demonstration endpoint"""
    
    @cache.cached(timeout=60)  # Cache for 1 minute
    def get(self):
        """Cached endpoint example"""
        import time
        
        # Simulate expensive operation
        time.sleep(2)
        
        return {
            'message': 'This response is cached for 60 seconds',
            'timestamp': datetime.utcnow().isoformat(),
            'data': 'Some expensive computed data'
        }
    
    def delete(self):
        """Clear specific cache"""
        cache.delete_memoized(self.get)
        return {'message': 'Cache cleared successfully'}

class TaskStatus(Resource):
    """Background task status endpoint"""
    
    @jwt_required()
    def get(self, task_id):
        """Get task status"""
        task = celery.AsyncResult(task_id)
        
        return {
            'task_id': task_id,
            'status': task.status,
            'result': task.result if task.ready() else None,
            'info': task.info
        }

class EmailSender(Resource):
    """Email sending endpoint"""
    
    @role_required(['admin', 'manager'])
    def post(self):
        """Send email (background task)"""
        try:
            data = request.get_json()
            
            required_fields = ['to_email', 'subject', 'body']
            for field in required_fields:
                if field not in data:
                    return {'message': f'{field} is required'}, 400
            
            # Send email as background task
            task = send_email_task.delay(
                data['to_email'],
                data['subject'],
                data['body']
            )
            
            return {
                'message': 'Email queued for sending',
                'task_id': task.id
            }, 202
            
        except Exception as e:
            return {'message': f'Email queuing failed: {str(e)}'}, 500

class Analytics(Resource):
    """Analytics and statistics endpoint"""
    
    @role_required(['admin', 'manager'])
    @cache.cached(timeout=1800)  # Cache for 30 minutes
    def get(self):
        """Get application analytics"""
        try:
            # User statistics
            total_users = User.query.count()
            active_users = User.query.filter_by(is_active=True).count()
            users_by_role = db.session.query(
                User.role, db.func.count(User.id)
            ).group_by(User.role).all()
            
            # Item statistics
            total_items = Item.query.filter(Item.status != 'deleted').count()
            items_by_status = db.session.query(
                Item.status, db.func.count(Item.id)
            ).group_by(Item.status).all()
            
            # Order statistics
            total_orders = Order.query.count()
            total_revenue = db.session.query(
                db.func.sum(Order.total_amount)
            ).scalar() or 0
            
            orders_by_status = db.session.query(
                Order.status, db.func.count(Order.id)
            ).group_by(Order.status).all()
            
            return {
                'users': {
                    'total': total_users,
                    'active': active_users,
                    'by_role': dict(users_by_role)
                },
                'items': {
                    'total': total_items,
                    'by_status': dict(items_by_status)
                },
                'orders': {
                    'total': total_orders,
                    'total_revenue': total_revenue,
                    'by_status': dict(orders_by_status)
                },
                'generated_at': datetime.utcnow().isoformat()
            }
            
        except Exception as e:
            return {'message': f'Analytics generation failed: {str(e)}'}, 500

# ==========================
# API Routes Registration
# ==========================

# Health and utility
api.add_resource(HealthCheck, '/api/health')
api.add_resource(CacheDemo, '/api/cache-demo')
api.add_resource(Analytics, '/api/analytics')

# Authentication
api.add_resource(AuthRegister, '/api/auth/register')
api.add_resource(AuthLogin, '/api/auth/login')
api.add_resource(AuthLogout, '/api/auth/logout')
api.add_resource(UserProfile, '/api/auth/profile')

# User management
api.add_resource(UserManagement, '/api/users', '/api/users/<int:user_id>')

# Category management
api.add_resource(CategoryAPI, '/api/categories', '/api/categories/<int:category_id>')

# Item management
api.add_resource(ItemAPI, '/api/items', '/api/items/<int:item_id>')

# Order management
api.add_resource(OrderAPI, '/api/orders', '/api/orders/<int:order_id>')

# File handling
api.add_resource(FileUpload, '/api/upload', '/api/files/<filename>')

# Background tasks
api.add_resource(TaskStatus, '/api/tasks/<task_id>')
api.add_resource(EmailSender, '/api/send-email')

# ==========================
# Error Handlers
# ==========================

@app.errorhandler(404)
def not_found(error):
    return jsonify({'message': 'Resource not found'}), 404

@app.errorhandler(500)
def internal_error(error):
    db.session.rollback()
    return jsonify({'message': 'Internal server error'}), 500

@app.errorhandler(ValidationError)
def validation_error(error):
    return jsonify({'message': 'Validation error', 'errors': error.messages}), 400

# ==========================
# Database Initialization
# ==========================

def create_admin_user():
    """Create default admin user"""
    with app.app_context():
        # Create tables
        db.create_all()
        
        # Check if admin exists
        admin = User.query.filter_by(role='admin').first()
        if not admin:
            admin = User(
                username='admin',
                email='admin@example.com',
                role='admin',
                is_active=True,
                is_approved=True
            )
            admin.set_password('admin123')  # Change this in production!
            
            db.session.add(admin)
            db.session.commit()
            print("✅ Default admin user created (admin@example.com / admin123)")
        
        # Create sample categories
        if not Category.query.first():
            categories = [
                Category(name='Electronics', description='Electronic items and gadgets'),
                Category(name='Books', description='Books and educational materials'),
                Category(name='Clothing', description='Apparel and accessories'),
                Category(name='Tasks', description='Work tasks and assignments')
            ]
            
            for category in categories:
                db.session.add(category)
            
            db.session.commit()
            print("✅ Sample categories created")

# ==========================
# CLI Commands
# ==========================

@app.cli.command()
def init_db():
    """Initialize database with sample data"""
    create_admin_user()
    print("✅ Database initialized successfully!")

@app.cli.command()
def clear_cache():
    """Clear all cache"""
    cache.clear()
    redis_client.flushdb()
    print("✅ Cache cleared successfully!")

@app.cli.command()
def create_sample_data():
    """Create sample data for testing"""
    with app.app_context():
        # Create sample users
        if User.query.count() <= 1:  # Only admin exists
            users = [
                {'username': 'manager1', 'email': 'manager@example.com', 'role': 'manager'},
                {'username': 'employee1', 'email': 'employee@example.com', 'role': 'employee'},
                {'username': 'customer1', 'email': 'customer@example.com', 'role': 'customer'}
            ]
            
            for user_data in users:
                user = User(**user_data, is_active=True, is_approved=True)
                user.set_password('password123')
                db.session.add(user)
            
            db.session.commit()
            print("✅ Sample users created")
        
        # Create sample items
        if Item.query.count() == 0:
            category = Category.query.first()
            admin = User.query.filter_by(role='admin').first()
            
            items = [
                {
                    'title': 'Sample Product 1',
                    'description': 'This is a sample product for testing',
                    'price': 99.99,
                    'category_id': category.id if category else None,
                    'owner_id': admin.id
                },
                {
                    'title': 'Sample Task',
                    'description': 'Complete the project documentation',
                    'priority': 'high',
                    'due_date': datetime.utcnow() + timedelta(days=7),
                    'category_id': category.id if category else None,
                    'owner_id': admin.id
                }
            ]
            
            for item_data in items:
                item = Item(**item_data)
                db.session.add(item)
            
            db.session.commit()
            print("✅ Sample items created")

# ==========================
# Application Factory
# ==========================

def create_app(config_name='development'):
    """Application factory pattern"""
    # This structure allows for different configurations
    # You can extend this for testing, production, etc.
    
    if config_name == 'testing':
        app.config['TESTING'] = True
        app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
    elif config_name == 'production':
        app.config['DEBUG'] = False
        # Add production-specific configurations
    
    return app

# ==========================
# Application Entry Point
# ==========================

if __name__ == '__main__':
    # Initialize database on startup
    create_admin_user()
    
    print("""
    🚀 Flask Boilerplate Application Started!
    
    📖 Available Endpoints:
    ├── GET  /api/health                 - Health check
    ├── POST /api/auth/register          - User registration
    ├── POST /api/auth/login             - User login
    ├── POST /api/auth/logout            - User logout
    ├── GET  /api/auth/profile           - Get user profile
    ├── GET  /api/users                  - Get all users (admin)
    ├── GET  /api/categories             - Get categories
    ├── POST /api/categories             - Create category (admin/manager)
    ├── GET  /api/items                  - Get items (cached)
    ├── POST /api/items                  - Create item
    ├── GET  /api/orders                 - Get orders
    ├── POST /api/orders                 - Create order
    ├── POST /api/upload                 - Upload file
    ├── POST /api/send-email             - Send email (admin/manager)
    ├── GET  /api/analytics              - Get analytics (admin/manager)
    └── GET  /api/cache-demo             - Cache demonstration
    
    🔐 Default Admin Credentials:
    Email: admin@example.com
    Password: admin123
    
    📝 CLI Commands:
    ├── flask init-db                    - Initialize database
    ├── flask clear-cache                - Clear all cache
    └── flask create-sample-data         - Create sample data
    
    🎯 Project Customization Ideas:
    ├── Quiz System: Item = Question, Category = Subject, Order = Quiz Attempt
    ├── Parking System: Item = Parking Spot, Category = Zone, Order = Booking
    ├── Task Management: Item = Task, Category = Project, Order = Sprint
    ├── E-commerce: Item = Product, Category = Product Category, Order = Purchase
    ├── Event Management: Item = Event, Category = Event Type, Order = Registration
    └── Inventory System: Item = Product, Category = Warehouse Section, Order = Stock Movement
    
    🔧 Background Tasks:
    ├── Email sending (Celery)
    ├── Order processing (Celery)
    ├── Data cleanup (Celery Beat - scheduled)
    └── Custom tasks (easily extendable)
    
    ⚡ Features:
    ├── JWT Authentication with role-based access
    ├── Redis caching with decorators
    ├── File upload with validation
    ├── Pagination and filtering
    ├── Background job processing
    ├── Email notifications
    ├── Analytics and reporting
    ├── Comprehensive error handling
    └── Database relationships and models
    """)
    
    # Run the application
    app.run(
        debug=True,
        host='0.0.0.0',
        port=5000
    )
 ```