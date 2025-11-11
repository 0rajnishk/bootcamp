---
layout: default
title: Day 3 - Frontend Foundation - Vue.js & Integration
nav_order: 4
---

# Day 3: Frontend Foundation - Vue.js & Integration

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


## Vue Router Setup
### Minimal frontend structure and files

Project tree (minimal):
```
3/frontend/
├─ index.html
├─ package.json
├─ vite.config.js
└─ src/
   ├─ main.js
   ├─ App.vue
   ├─ router/
   │  └─ index.js
   └─ Views/
      ├─ HomeView.vue
      ├─ LoginView.vue
      └─ SignUp.vue
```

index.html
```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vite App</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

src/main.js
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```

src/App.vue
```vue
<template><RouterView/></template>
```

src/router/index.js
```javascript
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../Views/HomeView.vue'
import Login from '../Views/LoginView.vue'
import Signup from '../Views/SignUp.vue'

export default createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    { path: '/', component: Home },
    { path: '/login', component: Login },
    { path: '/signup', component: Signup }
  ]
})
```

src/Views/HomeView.vue
```vue
<template>
    <div>
        <h1>Welcome to the Home View {{ data }}</h1>

        <p>{{ user.username }}</p>
        <p>{{ user.email }}</p>
        <p>{{ user.phone_number }}</p>
        <button @click="popAlert()">alert</button>
        <button @click="hello()">fetch hello</button>
    </div>
</template>

<script>
import axios from 'axios';

export default{

    data() {
        return{
            data:'data value',
            users:{}
        }
    },
    methods: {

        popAlert(){
            alert('Hello world! ' + this.data)
        },

        async hello(){
            const token = localStorage.getItem('token')
            const response = await axios.get('http://127.0.0.1:5000/register', {
                headers:{
                    Authorization: `Bearer ${token}`
                }
            });
            alert('api call ')
            console.log(response)
            this.users = response.data.user
            alert(response.data.msg)
        },
        
        
    },
    mounted() {
        this.hello()
    }

}

</script>


<style scoped>

</style>
```

src/Views/LoginView.vue
```vue
<template>
    <h1>Login</h1>
    <form>
        <input v-model="data.username" type="text" placeholder="username" />
        <input v-model="data.password" type="password" placeholder="password" />
        <button @click="login()" type="submit">login</button>

    </form>
</template>


<script>
import  axios from  'axios';

export default{
    data(){
        return {
            data:{}
        }
    },
    methods: {
        async login(){
            const response = await axios.post('http://127.0.0.1:5000/login', this.data);
            alert(response.data.msg)
            const token = response.data.token;
            localStorage.setItem('token', token);

        }
    }

}
</script>
```

src/Views/SignUp.vue
```vue
<template>
    <h1>Signup</h1>
    <form>
        <input v-model="data.username" type="text" placeholder="username" />
        <input v-model="data.email" type="email" placeholder="email" />
        <input v-model="data.phone_number" type="text" placeholder="phone number" />
        <input v-model="data.password" type="password" placeholder="password" />
        <button @click="register()" type="submit">Register</button>
    </form>


</template>

<script>
import axios from 'axios';


export default {

    data() {
        return {
            data1: 'data1 value',
            data: {
                username: '',
                email: '',
                phone_number: '',
                password: ''
            }

        }
    },
    methods: {

        async popAlert() {
            alert('Hello world!' + this.data1)
        },

        async register() {
            const response = await axios.post('http://127.0.0.1:5000/register', this.data);
            console.log(response)
            alert(response.data.msg)
        }


    }

}


</script>

```

## Quick component anatomy & common patterns (short examples)

data() (shape)
```javascript
data() {
    return {
        items: [],
        loading: false,
        form: { title: '', body: '' }
    }
}
```

components: (register local components)
```javascript
components: {
    ChildComp, // imported above
    AnotherComp
}
```

methods: (quick examples including axios calls)
```javascript
methods: {
    async fetchItems() {
        this.loading = true
        try {
            const res = await axios.get('http://127.0.0.1:5000/api/items')
            this.items = res.data
        } finally {
            this.loading = false
        }
    },

    async createItem(payload) {
        // POST
        const res = await axios.post('http://127.0.0.1:5000/api/items', payload)
        return res.data
    },

    async removeItem(id) {
        // DELETE
        await axios.delete(`http://127.0.0.1:5000/api/items/${id}`)
        this.items = this.items.filter(i => i.id !== id)
    }
}
```


Lifecycle hooks (examples)
```javascript
created() {
    // run before mounting — good for preparing data
    this.fetchItems()
},
mounted() {
    // DOM is ready — good for DOM-related setup
    console.log('component mounted')
},
beforeUnmount() {
    // cleanup
}
```
