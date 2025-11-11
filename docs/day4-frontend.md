---
layout: default
title: Day 4 - Frontend - Vue.js & Integration
nav_order: 4
---

# Day 4: Frontend - Vue.js & Integration

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
    ├─ components/
    │  └─ CompA.vue
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

        <!-- Child component: listen for the custom event 'send-data' emitted by CompA -->
        <CompA :msg="data" @send-data="ChildData" />

        <p>Child says: {{ dataA }}</p>

        <button @click="popAlert()">alert</button>
        <button @click="hello()">fetch hello</button>
    </div>
</template>

<script>
import axios from 'axios';
import CompA from '@/components/CompA.vue';

export default{

    data() {
        return{
            data:'data value',
            dataA: 'waiting..',
            users:{}
        }
    },
    components:{ 
        CompA
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
        ChildData(data) {
            // show and store the payload coming from CompA
            alert('received from child: ' + data)
            this.dataA = data
        }

        

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

src/components/CompA.vue
```vue
<template>
    <div class="comp-a">
        <h1>component A</h1>
        <h2>{{ msg }}</h2>
        <button @click="sendToParent">send data</button>
    </div>
</template>

<script>
export default {
    data() {
        return {
            data: 'data from child'
        }
    },
    props: {
        msg: String
    },

    methods: {
        sendToParent() {
            this.$emit('send-data', this.data)
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

Axios standalone examples (minimal)
```javascript
// GET
await axios.get('http://127.0.0.1:5000/api/items')

// POST
await axios.post('http://127.0.0.1:5000/api/items', { title: 'hello' })

// DELETE
await axios.delete('http://127.0.0.1:5000/api/items/123')
```

Parent → Child and Child → Parent (data flow)

Parent (template + handler)
```html
<!-- Parent template -->
<ChildComp :item="selectedItem" @update-item="onUpdateItem" />
```

```javascript
// Parent methods
methods: {
    onUpdateItem(newData) {
        // handle payload from child
        this.selectedItem = newData
    }
}
```

Child (props + emit)
```javascript
props: ['item'],
methods: {
    sendUpdate() {
        const payload = { ...this.item, updatedAt: Date.now() }
        this.$emit('update-item', payload)
    }
}
```

## Template directives (short examples)

v-if / v-else / v-else-if
```html
<div>
    <p v-if="items.length === 0">No items</p>
    <p v-else-if="loading">Loading...</p>
    <p v-else>Found {{ items.length }} items</p>
</div>
```

v-show (toggle visibility without re-render)
```html
<div v-show="isVisible">This is visible but not removed from DOM when false</div>
```

v-for (lists)
```html
<ul>
    <li v-for="(item, idx) in items" :key="item.id">{{ idx + 1 }}. {{ item.title }}</li>
    <!-- shorthand for binding: :title="item.title" -->
</ul>
```

v-bind and v-on (shorthands)
```html
<img :src="imageUrl" :alt="imageAlt" />
<button @click="handleClick">Click me</button>
```

Using expressions and event args
```html
<button @click="addItem('new')">Add</button>
<input v-model="form.title" />
```

Computed property example (short)
```javascript
computed: {
    itemsCount() {
        return this.items.length
    }
}
```

Watch example (short)
```javascript
watch: {
    'form.title'(newVal) {
        console.log('title changed', newVal)
    }
}
```