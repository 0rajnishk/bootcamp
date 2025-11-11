---
title: "Bootcamp Tutorial 5: Flask, Celery, Redis, and SMTP Integration"  
date: "2025-03-02 00:00:00"
categories: [bootcamp]
tags: [bootcamp, vue, flask]
---

---

In this tutorial, we integrate **Flask**, **Celery**, **Redis**, **Flask-Caching**, and **SMTP** to build an app that supports:

- Sending emails via SMTP
- Caching API responses
- Clearing cache dynamically
- Background task queuing
- Scheduled tasks with Celery Beat

---

## Prerequisites

Install required dependencies:

```bash
pip install Flask Flask-RESTful Celery redis Flask-Mail Flask-Caching python-dotenv
```

Ensure Redis is installed and running:

```bash
sudo apt update
sudo apt install redis-server
sudo systemctl start redis
```

---

## Setup: Flask Application

Create a file `main.py`:

```python
import os
from flask import Flask, request
from flask_restful import Api, Resource
from flask_mail import Mail, Message
from flask_caching import Cache
from celery import Celery
from dotenv import load_dotenv
from celery.schedules import crontab

# Load .env
load_dotenv()

app = Flask(__name__)
api = Api(app)

# ============================
# Flask-Mail Configuration
# ============================
app.config['MAIL_SERVER'] = 'smtp.gmail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.getenv('MAIL_USER')
app.config['MAIL_PASSWORD'] = os.getenv('MAIL_PASS')
app.config['MAIL_DEFAULT_SENDER'] = os.getenv('MAIL_USER')
mail = Mail(app)

# ============================
# Flask-Caching with Redis
# ============================
app.config['CACHE_TYPE'] = 'RedisCache'
app.config['CACHE_REDIS_HOST'] = 'localhost'
app.config['CACHE_REDIS_PORT'] = 6379
app.config['CACHE_REDIS_DB'] = 1
app.config['CACHE_REDIS_URL'] = 'redis://localhost:6379/1'
app.config['CACHE_DEFAULT_TIMEOUT'] = 60
cache = Cache(app)

# ============================
# Celery Configuration
# ============================
app.config['broker_url'] = os.getenv('BROKER_URL', 'redis://localhost:6379/0')
app.config['result_backend'] = os.getenv('RESULT_BACKEND', 'redis://localhost:6379/0')

celery = Celery(app.name, broker=app.config['broker_url'], backend=app.config['result_backend'])
celery.conf.broker_connection_retry_on_startup = True

# ============================
# Celery Context Setup
# ============================
def init_celery(flask_app):
    celery_app = Celery(
        flask_app.import_name,
        broker=flask_app.config['broker_url'],
        backend=flask_app.config['result_backend']
    )
    celery_app.conf.update(flask_app.config)

    class ContextTask(celery_app.Task):
        def __call__(self, *args, **kwargs):
            with flask_app.app_context():
                return super().__call__(*args, **kwargs)

    celery_app.Task = ContextTask
    return celery_app

celery = init_celery(app)
```

---

## 1. Send Emails with SMTP

```python
class SendEmail(Resource):
    def get(self):
        email = request.args.get('email')
        if not email:
            return {'message': 'Error: No email provided'}, 400

        msg = Message(
            subject="Test Email from Flask",
            recipients=[email],
            body="This is a test email sent via Flask and SMTP."
        )

        try:
            mail.send(msg)
            return {'message': f'Email sent to {email}!'}, 200
        except Exception as e:
            return {'message': f"Error: {str(e)}"}, 500

api.add_resource(SendEmail, '/send-email')
```

Test:

```bash
curl "http://localhost:5000/send-email?email=you@example.com"
```

---

## 2. Caching with Flask-Caching

```python
class CacheDemo(Resource):
    @cache.cached()
    def get(self):
        return {'data': 'This is a cached response!'}, 200

api.add_resource(CacheDemo, '/cache')
```

### Clear Cache

```python
class DeleteCache(Resource):
    def post(self):
        cache.clear()
        return {'message': 'Cache cleared successfully!'}, 200

api.add_resource(DeleteCache, '/delete-cache')
```

Test:

```bash
curl http://localhost:5000/cache
curl -X POST http://localhost:5000/delete-cache
```

---

## 3. Background Task with Celery

```python
@celery.task(name="tasks.background_task")
def background_task():
    return 'Background task completed'

class QueuedTask(Resource):
    def get(self):
        task = background_task.apply_async()
        return {'task_id': task.id}, 202

api.add_resource(QueuedTask, '/queued-task')
```

---

## 4. Schedule Daily Task with Celery Beat

```python
celery.conf.timezone = 'Asia/Kolkata'
celery.conf.beat_schedule = {
    'daily-reminder': {
        'task': 'tasks.send_reminders',
        'schedule': crontab(hour=7, minute=0),
    },
}

@celery.task(name="tasks.send_reminders")
def send_reminders():
    print("Reminder sent at 7:00 AM IST!")
    return "Reminder triggered"
```

---

## .env File Example

```
MAIL_USER=your_email@gmail.com
MAIL_PASS=your_email_password
BROKER_URL=redis://localhost:6379/0
RESULT_BACKEND=redis://localhost:6379/0
```

---

## Running the App

```bash
# 1. Run Flask
python3 main.py

# 2. Start Celery Worker
celery -A main.celery worker --loglevel=info

# 3. Start Celery Beat
celery -A main.celery beat --loglevel=info
```

---

## Test Endpoints

- **Send Email:** `GET /send-email?email=you@example.com`
- **Cache Demo:** `GET /cache`
- **Clear Cache:** `POST /delete-cache`
- **Queue Task:** `GET /queued-task`
- **Daily Reminder:** Check logs after 7:00 AM IST

---
