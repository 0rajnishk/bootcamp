---
layout: default
title: bootcamp day 1- Introduction
nav_order: 1
---


# Setting Up Things for the Project

Follow the instructions given in the below site to install WSL:

[WSL Installation Guide](https://learn.microsoft.com/en-us/windows/wsl/install)

Make sure you reboot your system after installing WSL.

## Now Install Git in Your WSL Setup

```bash
sudo apt install git
git config --global user.name "Your Name"
git config --global user.email "Your Email"
```

## install github cli(gh) for linux/wsl only
```bash
## Step 1: Add the GitHub CLI repository
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key C99B11DEB97541F0
sudo apt-add-repository https://cli.github.com/packages
## Step 2: Update package lists
sudo apt update
## Step 3: Install GitHub CLI
sudo apt install gh

gh --version
```

## github login
```bash
    gh auth login --web
```


## create a repository on github
## clone repository from github
```bash
    git clone <repository url>
```




## Download and Install Node and npm

Download and install NVM (Node Version Manager):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

Then, install Node.js:

```bash
nvm install 22
```

Verify the Node.js version:

```bash
node -v  # Should print "v22.13.1"
nvm current  # Should print "v22.13.1"
```

Verify npm version:

```bash
npm -v  # Should print "10.9.2"
```

Check your Node.js and npm version:

```bash
node -v
npm -v
```

Check your Python version (ensure you have Python installed and it's above 3.6):

```bash
python3 --version
```

# Initializing the Frontend and Backend Project

## Setting Up Frontend

Go to the root folder of your project:

```bash
cd /path/to/your/project
```




Create the Vue frontend project using Vite:

```bash
npm create vite@latest frontend --template vue
cd frontend
npm install
npm run dev
```

## Setting Up Backend

Create the backend directory and set up a Python virtual environment:

```bash
mkdir backend
cd backend
python3 -m venv .env
```


Activate virtual environment:

```bash
source .env/bin/activate
```


## Push Your New Changes to the Remote Origin

Commit your changes:

```bash
git commit -m "Initial commit"
git push
```

---

# **1️⃣ Setting Up Vue Project Structure**
A typical Vue.js project contains the following key files:

```
📂 vue-auth-project/
 ├── 📂 src/
 │   ├── 📂 components/         # Reusable components (optional)
 │   ├── 📂 views/              # Main pages (Login, Signup, Home)
 │   ├── 📂 router/             # Vue Router Configuration
 │   │   ├── index.js
 │   ├── 📂 assets/             # Static assets (images, CSS)
 │   ├── App.vue                # Root Vue component
 │   ├── main.js                # Main entry file
 ├── index.html                 # Base HTML file
 ├── vite.config.js             # Vite configuration (for Vue)
 ├── package.json               # Dependencies & scripts
```

---

# **2️⃣ Configuring Vue.js with Routing**
### ** `main.js` - Registering Vue App & Router**
 `src/main.js`
```javascript
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';

const app = createApp(App);
app.use(router);
app.mount('#app');
```
**This initializes the Vue application** and registers the **router**.

---

### ** `App.vue` - Root Component**
 `src/App.vue`
```vue
<template>
  <RouterView />
</template>
```
 **`<RouterView />` is a placeholder for dynamically loaded pages.**  
- It loads different pages based on the **URL path**.

---

### ** `index.html` - The Main Entry Point**
 `public/index.html`
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link rel="icon" type="image/svg+xml" href="/vite.svg" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Vite + Vue</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="/src/main.js"></script>
</body>
</html>
```
 **Vue renders the app inside `<div id="app"></div>`.**

---

# **3️⃣ Setting Up Vue Router**
### ** `router/index.js` - Defining Routes**
 `src/router/index.js`
```javascript
import { createRouter, createWebHistory } from 'vue-router';
import Home from '../views/HomeView.vue';
import Login from '../views/LoginView.vue';
import Signup from '../views/SignupView.vue';

const router = createRouter({
    history: createWebHistory(import.meta.env.BASE_URL),
    routes: [
        { path: '/', name: 'home', component: Home },
        { path: '/login', name: 'login', component: Login },
        { path: '/signup', name: 'signup', component: Signup }
    ]
});

export default router;
```

 **Key Features:**
- Uses **`createWebHistory()`** for clean URLs (`/login` instead of `#/login`).
- Defines **routes** (`/`, `/login`, `/signup`).
- Each route loads a **view component** dynamically.

---