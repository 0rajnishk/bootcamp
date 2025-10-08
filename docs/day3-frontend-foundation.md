---
layout: default
title: Day 3 - Frontend Foundation - Vue.js & Integration
nav_order: 4
---

# Day 3: Frontend Foundation - Vue.js & Integration

## Learning Objectives
By the end of this day, you will be able to:
- Set up a Vue.js project with Vite
- Understand Vue.js Options API basics
- Create and structure Vue components
- Connect frontend to Flask API using Axios
- Display data from the backend
- Handle basic user interactions

## Prerequisites
- Completed Day 2 exercises
- Node.js installed (version 16+)
- Understanding of HTML, CSS, and JavaScript basics
- Working Flask API from Day 2

## What is Vue.js?

### Vue.js Overview
- Progressive JavaScript framework
- Easy to learn and integrate
- Component-based architecture
- Reactive data binding
- Great for building user interfaces

### Vue.js?
- Simple syntax
- Excellent documentation
- Small learning curve
- Flexible and lightweight
- Great developer experience

## Setting Up Vue.js with Vite

### Installation
```bash
# Create new Vue project
npm create vue@latest task-manager-frontend

# Navigate to project directory
cd task-manager-frontend

# Install dependencies
npm install

# Install additional packages
npm install axios vue-router@4
```
find detailed guid to install vue.js with vite [here](https://0rajnishk.github.io/bootcamp/docs/day0-environment-setup.html#2-create-a-new-vue-project)

### Project Structure (views-only routing, no App.vue)
```
3/
├── frontend/
│   ├── src/
│   │   ├── views/
│   │   │   ├── HomeView.vue
│   │   │   ├── LoginView.vue
│   │   │   └── RegisterView.vue
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── router/
│   │   │   └── index.js
│   │   └── main.js
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
└── backend/ (Day 2 code kept running)
```

## Vue.js Options API Basics

### Component Structure
```vue
<template>
  <!-- HTML template -->
  <div>
    <h1>{{ title }}</h1>
    <button @click="handleClick">Click me</button>
  </div>
</template>

<script>
export default {
  name: 'MyComponent',
  data() {
    return {
      title: 'Hello Vue!',
      count: 0
    }
  },
  methods: {
    handleClick() {
      this.count++
    }
  },
  computed: {
    doubleCount() {
      return this.count * 2
    }
  }
}
</script>

<style scoped>
/* Component-specific styles */
h1 {
  color: blue;
}
</style>
```

### Understanding the Options API

#### Data
- Reactive data properties
- Automatically updates the template when changed
- Must be a function that returns an object

#### Methods
- Functions that can be called from the template
- Access component data with `this.propertyName`
- Handle user interactions

#### Computed Properties
- Derived values based on other data
- Cached and only re-evaluate when dependencies change
- More efficient than methods for derived data

## Root setup (no App.vue)

### index.html
Ensure your Vite `index.html` has a root element:
```html
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Day 3 </title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
  </html>
```

### main.js (renders RouterView directly)
```javascript
// src/main.js
import { createApp, h } from 'vue'
import router from './router'

// Minimal root component that only renders the current route
const Root = {
  render() {
    return h('div', { id: 'app', style: 'font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px;' }, [
      h('nav', { style: 'display:flex; justify-content:space-between; align-items:center; margin-bottom:30px; padding-bottom:20px; border-bottom:1px solid #eee;' }, [
        h('h1', 'Task Manager'),
        h('div', [
          h('a', { href: '/', onClick: (e) => { e.preventDefault(); router.push('/'); } }, 'Home'),
          ' | ',
          h('a', { href: '/login', onClick: (e) => { e.preventDefault(); router.push('/login'); } }, 'Login'),
          ' | ',
          h('a', { href: '/register', onClick: (e) => { e.preventDefault(); router.push('/register'); } }, 'Register')
        ])
      ]),
      h('main', [h('router-view')])
    ])
  }
}

createApp(Root).use(router).mount('#app')
```

## API Integration with Axios

### Basic Axios Usage Example
Here's a simple example of how to use axios to call your backend APIs:

```javascript
// Basic axios usage
async getHello(){
  const response = await axios.get('http://127.0.0.1:5000/')
  alert(response.data)
}
```

This example shows:
- Making a GET request to your backend
- Using `await` for asynchronous operations
- Accessing response data with `response.data`
- The response will be `'Hello, World!'` from your backend

### Backend APIs Available
Your Day 2 backend provides these endpoints:

1. **GET /** - Public endpoint
   - Returns: `'Hello, World!'`
   - No authentication required

2. **POST /** - Protected endpoint
   - Returns: `'Hello, World!'`
   - Requires JWT token in Authorization header
   - Uses `@jwt_required()` decorator

3. **POST /signup** - User registration
   - Body: `{ full_name, email, password }`
   - Returns: `'user registered successfully'`

4. **POST /login** - User authentication
   - Body: `{ email, password }`
   - Returns: `{ token: 'jwt_token_here' }`

### Setting Up Axios (wired to Day 2 API)
```javascript
// src/services/api.js
import axios from 'axios'

const API_BASE_URL = 'http://127.0.0.1:5000'

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: { 'Content-Type': 'application/json' }
})

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token')
      localStorage.removeItem('email')
    }
    return Promise.reject(error)
  }
)

export const authService = {
  async register(full_name, email, password) {
    const res = await api.post('/signup', { full_name, email, password })
    return res.data
  },
  async login(email, password) {
    const res = await api.post('/login', { email, password })
    return res.data // { token }
  }
}

export const helloService = {
  async getHello() {
    const res = await api.get('/')
    return res.data // 'Hello, World!'
  },
  async postHello() {
    const res = await api.post('/')
    return res.data // 'Hello, World!' (requires JWT)
  }
}
```

## Creating Components

### LoginForm Component
```vue
<template>
  <div class="login-form">
    <h2>Login</h2>
    <form @submit.prevent="handleLogin">
      <div>
        <label>Username:</label>
        <input v-model="username" type="text" required>
      </div>
      <div>
        <label>Password:</label>
        <input v-model="password" type="password" required>
      </div>
      <button type="submit" :disabled="loading">
        {{ loading ? 'Logging in...' : 'Login' }}
      </button>
    </form>
    
    <div v-if="error" class="error">
      {{ error }}
    </div>
  </div>
</template>

<script>
import { authService } from '../services/api.js'

export default {
  name: 'LoginForm',
  data() {
    return {
      username: '',
      password: '',
      loading: false,
      error: ''
    }
  },
  methods: {
    async handleLogin() {
      this.loading = true
      this.error = ''
      
      try {
        const response = await authService.login(this.username, this.password)
        this.$emit('login-success', response)
      } catch (error) {
        this.error = error.response?.data?.message || 'Login failed'
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
.login-form {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

input {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.error {
  color: red;
  margin-top: 10px;
}
</style>
```

### TaskList Component
```vue
<template>
  <div class="task-list">
    <h2>My Tasks</h2>
    
    <div v-if="loading" class="loading">
      Loading tasks...
    </div>
    
    <div v-else-if="tasks.length === 0" class="no-tasks">
      No tasks found. Create your first task!
    </div>
    
    <div v-else>
      <div v-for="task in tasks" :key="task.id" class="task-item">
        <h3>{{ task.title }}</h3>
        <p v-if="task.description">{{ task.description }}</p>
        <span :class="['status', task.completed ? 'completed' : 'pending']">
          {{ task.completed ? 'Completed' : 'Pending' }}
        </span>
        <div class="actions">
          <button @click="toggleTask(task)" class="toggle-btn">
            {{ task.completed ? 'Mark Pending' : 'Mark Complete' }}
          </button>
          <button @click="deleteTask(task.id)" class="delete-btn">
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { taskService } from '../services/api.js'

export default {
  name: 'TaskList',
  data() {
    return {
      tasks: [],
      loading: true
    }
  },
  async mounted() {
    await this.loadTasks()
  },
  methods: {
    async loadTasks() {
      try {
        this.tasks = await taskService.getTasks()
      } catch (error) {
        console.error('Failed to load tasks:', error)
      } finally {
        this.loading = false
      }
    },
    
    async toggleTask(task) {
      try {
        await taskService.updateTask(task.id, {
          ...task,
          completed: !task.completed
        })
        task.completed = !task.completed
      } catch (error) {
        console.error('Failed to update task:', error)
      }
    },
    
    async deleteTask(taskId) {
      if (confirm('Are you sure you want to delete this task?')) {
        try {
          await taskService.deleteTask(taskId)
          this.tasks = this.tasks.filter(task => task.id !== taskId)
        } catch (error) {
          console.error('Failed to delete task:', error)
        }
      }
    }
  }
}
</script>

<style scoped>
.task-item {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 15px;
}

.status {
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: bold;
}

.status.completed {
  background: #d4edda;
  color: #155724;
}

.status.pending {
  background: #fff3cd;
  color: #856404;
}

.actions {
  margin-top: 10px;
}

.toggle-btn {
  background: #28a745;
  margin-right: 10px;
}

.delete-btn {
  background: #dc3545;
}

.loading, .no-tasks {
  text-align: center;
  padding: 40px;
  color: #666;
}
</style>
```

## Vue Router Setup

### Router Configuration
```javascript
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'
import LoginView from '../views/LoginView.vue'
import RegisterView from '../views/RegisterView.vue'

const routes = [
  { path: '/', name: 'Home', component: HomeView },
  { path: '/login', name: 'Login', component: LoginView },
  { path: '/register', name: 'Register', component: RegisterView }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

## Views

### HomeView.vue
```vue
<template>
  <div class="home">
    <h1>Welcome to Task Manager</h1>
    <p>Test the API integration with your backend using Axios!</p>
    
    <div class="api-demo">
      <h2>API Demo</h2>
      
      <!-- Public API Call -->
      <div class="api-section">
        <h3>Public API Call (GET /)</h3>
        <button @click="getHello" :disabled="loading.getHello">
          {{ loading.getHello ? 'Loading...' : 'Get Hello' }}
        </button>
        <div v-if="messages.getHello" class="message success">
          {{ messages.getHello }}
        </div>
      </div>
      
      <!-- Protected API Call -->
      <div class="api-section">
        <h3>Protected API Call (POST /)</h3>
        <p class="info">This requires JWT authentication. Login first to test this endpoint.</p>
        <button @click="postHello" :disabled="loading.postHello">
          {{ loading.postHello ? 'Loading...' : 'Post Hello (Protected)' }}
        </button>
        <div v-if="messages.postHello" class="message success">
          {{ messages.postHello }}
        </div>
        <div v-if="errors.postHello" class="message error">
          {{ errors.postHello }}
        </div>
      </div>
      
      <!-- Authentication Status -->
      <div class="auth-status">
        <h3>Authentication Status</h3>
        <div v-if="isAuthenticated" class="message success">
          ✅ You are logged in! Token: {{ token ? 'Present' : 'Missing' }}
        </div>
        <div v-else class="message warning">
          ⚠️ You are not logged in. <router-link to="/login">Login here</router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { helloService } from '../services/api.js'

export default {
  name: 'Home',
  data() {
    return {
      loading: {
        getHello: false,
        postHello: false
      },
      messages: {
        getHello: '',
        postHello: ''
      },
      errors: {
        postHello: ''
      },
      token: localStorage.getItem('token')
    }
  },
  computed: {
    isAuthenticated() {
      return !!this.token
    }
  },
  methods: {
    async getHello() {
      this.loading.getHello = true
      this.messages.getHello = ''
      
      try {
        const response = await helloService.getHello()
        this.messages.getHello = `Response: ${response}`
      } catch (error) {
        this.messages.getHello = `Error: ${error.response?.data || error.message}`
      } finally {
        this.loading.getHello = false
      }
    },
    
    async postHello() {
      this.loading.postHello = true
      this.messages.postHello = ''
      this.errors.postHello = ''
      
      try {
        const response = await helloService.postHello()
        this.messages.postHello = `Response: ${response}`
      } catch (error) {
        if (error.response?.status === 401) {
          this.errors.postHello = 'Unauthorized! Please login first.'
        } else {
          this.errors.postHello = `Error: ${error.response?.data || error.message}`
        }
      } finally {
        this.loading.postHello = false
      }
    }
  }
}
</script>
```

### LoginView.vue
```vue
<template>
  <div class="login-form">
    <h2>Login</h2>
    <form @submit.prevent="handleLogin">
      <div>
        <label>Email:</label>
        <input v-model="email" type="email" required />
      </div>
      <div>
        <label>Password:</label>
        <input v-model="password" type="password" required />
      </div>
      <button type="submit" :disabled="loading">{{ loading ? 'Logging in...' : 'Login' }}</button>
    </form>
    <p v-if="error" class="error">{{ error }}</p>
    <p v-if="success" class="success">{{ success }}</p>
  </div>
</template>

<script>
import { authService } from '../services/api'

export default {
  name: 'LoginView',
  data() {
    return { 
      email: '', 
      password: '', 
      loading: false, 
      error: '',
      success: ''
    }
  },
  methods: {
    async handleLogin() {
      this.loading = true
      this.error = ''
      this.success = ''
      
      try {
        const response = await authService.login(this.email, this.password)
        
        // Handle different response types
        if (response.token) {
          localStorage.setItem('token', response.token)
          localStorage.setItem('email', this.email)
          this.success = 'Login successful! Redirecting...'
          setTimeout(() => {
            this.$router.push('/')
          }, 1000)
        } else {
          this.error = 'Invalid response from server'
        }
      } catch (e) {
        this.error = e.response?.data || 'Login failed'
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
.login-form { 
  max-width: 400px; 
  margin: 0 auto; 
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

input {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  background: #007bff;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 4px;
  cursor: pointer;
  width: 100%;
}

button:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.error { 
  color: red; 
  margin-top: 10px; 
}

.success {
  color: green;
  margin-top: 10px;
}
</style>
```

### RegisterView.vue
```vue
<template>
  <div class="register-form">
    <h2>Register</h2>
    <form @submit.prevent="handleRegister">
      <div>
        <label>Full Name:</label>
        <input v-model="full_name" type="text" required />
      </div>
      <div>
        <label>Email:</label>
        <input v-model="email" type="email" required />
      </div>
      <div>
        <label>Password:</label>
        <input v-model="password" type="password" required />
      </div>
      <button type="submit" :disabled="loading">{{ loading ? 'Registering...' : 'Register' }}</button>
    </form>
    <p v-if="message" class="success">{{ message }}</p>
    <p v-if="error" class="error">{{ error }}</p>
  </div>
</template>

<script>
import { authService } from '../services/api'

export default {
  name: 'RegisterView',
  data() {
    return { full_name: '', email: '', password: '', loading: false, message: '', error: '' }
  },
  methods: {
    async handleRegister() {
      this.loading = true
      this.message = ''
      this.error = ''
      try {
        await authService.register(this.full_name, this.email, this.password)
        this.message = 'Registration successful. You can now login.'
      } catch (e) {
        this.error = e.response?.data?.message || 'Registration failed'
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
.register-form { max-width: 400px; margin: 0 auto; }
.success { color: green; margin-top: 10px; }
.error { color: red; margin-top: 10px; }
</style>
```

## Run Instructions

1. Start backend (Day 2):
```bash
python app.py
```

2. Start frontend (Day 3):
```bash
npm run dev
```

3. Navigate routes:
- `http://localhost:5173/` → `HomeView.vue` (test both GET and POST `/` endpoints)
- `http://localhost:5173/login` → `LoginView.vue` (stores JWT token)
- `http://localhost:5173/register` → `RegisterView.vue` (user registration)

### Testing the API Integration
1. **Test Public API**: Click "Get Hello" button on home page
2. **Test Registration**: Go to `/register` and create a new user
3. **Test Login**: Go to `/login` and authenticate with your credentials
4. **Test Protected API**: After login, return to home and click "Post Hello (Protected)"

### Main.js Setup
```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(router)
app.mount('#app')
```

## Hands-On Exercises

### Exercise 1: Basic Vue Setup
1. Create a new Vue project with Vite
2. Set up the basic project structure
3. Create a simple component that displays "Hello Vue!"

### Exercise 2: API Integration
1. Install and configure Axios
2. Create API service functions
3. Test API connection with your Flask backend

### Exercise 3: Login Component
1. Create a login form component
2. Implement authentication flow
3. Store JWT token in localStorage

### Exercise 4: Task Display
1. Create a component to display tasks
2. Fetch tasks from the API
3. Handle loading and error states

## Key Concepts Summary

### Vue.js Concepts
- **Components**: Reusable UI pieces
- **Data Binding**: Automatic synchronization between data and template
- **Methods**: Functions that handle user interactions
- **Lifecycle Hooks**: Functions that run at specific times (mounted, created, etc.)

### API Integration
- **Axios**: HTTP client for making API requests
- **Interceptors**: Middleware for requests and responses
- **Error Handling**: Proper error management
- **Authentication**: Token-based authentication

### Best Practices
- Keep components focused and reusable
- Use proper error handling
- Store sensitive data securely
- Follow Vue.js style guide
- Use meaningful component names

## Common Issues and Solutions

### Issue: CORS errors
**Solution**: Configure Flask-CORS in your backend or use a proxy in Vite config

### Issue: API calls not working
**Solution**: Check that the backend is running and the API URL is correct

### Issue: Token not being sent
**Solution**: Verify that the Authorization header is being set correctly

## Next Steps
Tomorrow we'll learn how to:
- Create forms for adding new tasks
- Implement two-way data binding
- Handle form validation
- Add computed properties for data manipulation

## Resources
- [Vue.js Documentation](https://vuejs.org/)
- [Vite Documentation](https://vitejs.dev/)
- [Axios Documentation](https://axios-http.com/)
- [Vue Router Documentation](https://router.vuejs.org/)
