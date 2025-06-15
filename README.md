Полное руководство по развертыванию Vue.js + Laravel
====================================================

**Внимание!** Это полное пошаговое руководство включает все этапы настройки - от установки ПО до готового работающего приложения.

1\. Установка необходимого ПО
-----------------------------

### 1.1 Установка XAMPP

1.  Скачайте XAMPP с [официального сайта](https://www.apachefriends.org/ru/index.html)
2.  Запустите установщик и выберите компоненты:
    *   Apache
    *   MySQL
    *   PHP
    *   phpMyAdmin
3.  Установите в `C:\xampp` (рекомендуется)
4.  После установки откройте панель управления XAMPP и запустите:
    *   Apache
    *   MySQL

### 1.2 Настройка хоста и виртуального хоста

#### 1.2.1 Редактирование hosts-файла

1.  Откройте файл `C:\Windows\System32\drivers\etc\hosts` от имени администратора
2.  Добавьте в конец файла:
    
    127.0.0.1 vue-laravel.local
    
3.  Сохраните файл

#### 1.2.2 Настройка виртуального хоста в Apache

1.  Откройте файл `C:\xampp\apache\conf\extra\httpd-vhosts.conf`
2.  Добавьте в конец:
    
    <VirtualHost \*:80>
        DocumentRoot "C:/xampp/htdocs/laravel-backend/public"
        ServerName vue-laravel.local
        <Directory "C:/xampp/htdocs/laravel-backend/public">
            AllowOverride All
            Require all granted
        </Directory>
    </VirtualHost>
            
    
3.  Сохраните файл и перезапустите Apache в панели XAMPP

### 1.3 Установка Composer

1.  Скачайте Composer с [официального сайта](https://getcomposer.org/download/)
2.  Запустите установщик
3.  Проверьте установку в командной строке:
    
    composer --version
    

### 1.4 Установка Node.js

1.  Скачайте Node.js LTS версию с [официального сайта](https://nodejs.org/)
2.  Запустите установщик
3.  Проверьте установку:
    
    node -v
    npm -v
            
    

### 1.5 Установка Vue CLI

npm install -g @vue/cli

2\. Настройка бэкенда (Laravel)
-------------------------------

### 2.1 Создание Laravel проекта

1.  Откройте командную строку
2.  Перейдите в папку XAMPP:
    
    cd C:\\xampp\\htdocs
    
3.  Создайте новый проект:
    
    composer create-project laravel/laravel laravel-backend
    
4.  Перейдите в папку проекта:
    
    cd laravel-backend
    

### 2.2 Настройка базы данных

1.  Откройте phpMyAdmin (http://localhost/phpmyadmin)
2.  Создайте новую базу данных с именем `vue_laravel_db`
3.  В папке проекта откройте файл `.env`
4.  Настройте подключение к БД:
    
    DB\_DATABASE=vue\_laravel\_db
    DB\_USERNAME=root
    DB\_PASSWORD=
            
    

### 2.3 Установка Laravel Sanctum

composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\\Sanctum\\Providers\\SanctumServiceProvider"
php artisan migrate

### 2.4 Настройка CORS

1.  Откройте `config/cors.php`
2.  Измените настройки:
    
    'paths' => \['api/\*', 'sanctum/csrf-cookie'\],
    'allowed\_origins' => \['http://localhost:8080'\],
    'allowed\_methods' => \['\*'\],
    'allowed\_headers' => \['\*'\],
            
    

### 2.5 Создание модели User

php artisan make:model User -m

Откройте созданный файл миграции в `database/migrations/..._create_users_table.php` и добавьте:

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->timestamps();
});

Запустите миграции:

php artisan migrate

### 2.6 Создание контроллеров

#### AuthController

php artisan make:controller AuthController

Откройте `app/Http/Controllers/AuthController.php` и добавьте:

namespace App\\Http\\Controllers;

use Illuminate\\Http\\Request;
use App\\Models\\User;
use Illuminate\\Support\\Facades\\Hash;
use Illuminate\\Validation\\ValidationException;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $request->validate(\[
            'name' => 'required|string',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:6'
        \]);

        $user = User::create(\[
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password)
        \]);

        return response()->json(\['message' => 'User registered'\]);
    }

    public function login(Request $request)
    {
        $request->validate(\[
            'email' => 'required|email',
            'password' => 'required'
        \]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages(\[
                'email' => \['The provided credentials are incorrect.'\]
            \]);
        }

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json(\[
            'user' => $user,
            'token' => $token
        \]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();
        return response()->json(\['message' => 'Logged out'\]);
    }
}

#### PostController

php artisan make:controller PostController --resource

Откройте `app/Http/Controllers/PostController.php` и добавьте:

namespace App\\Http\\Controllers;

use App\\Models\\Post;
use Illuminate\\Http\\Request;

class PostController extends Controller
{
    public function index()
    {
        return Post::all();
    }

    public function store(Request $request)
    {
        $request->validate(\[
            'title' => 'required|string',
            'content' => 'required|string'
        \]);

        $post = Post::create(\[
            'title' => $request->title,
            'content' => $request->content,
            'user\_id' => auth()->id()
        \]);

        return response()->json($post, 201);
    }
}

### 2.7 Создание модели Post

php artisan make:model Post -m

Откройте созданный файл миграции и добавьте:

Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->foreignId('user\_id')->constrained();
    $table->timestamps();
});

Запустите миграции:

php artisan migrate

### 2.8 Настройка роутов

Откройте `routes/api.php` и добавьте:

use App\\Http\\Controllers\\AuthController;
use App\\Http\\Controllers\\PostController;
use Illuminate\\Support\\Facades\\Route;

Route::post('/register', \[AuthController::class, 'register'\]);
Route::post('/login', \[AuthController::class, 'login'\]);

Route::middleware('auth:sanctum')->group(function () {
    Route::post('/logout', \[AuthController::class, 'logout'\]);
    Route::apiResource('posts', PostController::class);
});

### 2.9 Запуск Laravel сервера

php artisan serve --port=8000

3\. Настройка фронтенда (Vue.js)
--------------------------------

### 3.1 Создание Vue проекта

1.  Откройте командную строку
2.  Перейдите в нужную папку (например, `C:\projects`)
3.  Создайте проект:
    
    vue create vue-frontend
    
4.  Выберите ручную настройку и добавьте:
    *   Babel
    *   Router
    *   Vuex
5.  Перейдите в папку проекта:
    
    cd vue-frontend
    
6.  Установите дополнительные зависимости:
    
    npm install axios
    

### 3.2 Структура проекта

src/
├── assets/
├── components/
├── router/
│   └── index.js
├── store/
│   └── index.js
├── views/
│   ├── Login.vue
│   ├── Register.vue
│   ├── Posts.vue
├── App.vue
└── main.js

### 3.3 Настройка хранилища (Vuex)

Откройте `src/store/index.js` и замените содержимое на:

import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    user: null,
    token: localStorage.getItem('token') || null
  },
  mutations: {
    SET\_USER(state, user) {
      state.user = user
    },
    SET\_TOKEN(state, token) {
      state.token = token
      localStorage.setItem('token', token)
    },
    LOGOUT(state) {
      state.user = null
      state.token = null
      localStorage.removeItem('token')
    }
  },
  actions: {
    async login({ commit }, credentials) {
      const response = await axios.post('http://localhost:8000/api/login', credentials)
      commit('SET\_USER', response.data.user)
      commit('SET\_TOKEN', response.data.token)
    },
    async register({ commit }, userData) {
      await axios.post('http://localhost:8000/api/register', userData)
    },
    logout({ commit }) {
      commit('LOGOUT')
    }
  }
})

### 3.4 Настройка роутера

Откройте `src/router/index.js` и замените содержимое на:

import Vue from 'vue'
import VueRouter from 'vue-router'
import Login from '../views/Login.vue'
import Register from '../views/Register.vue'
import Posts from '../views/Posts.vue'

Vue.use(VueRouter)

const routes = \[
  { path: '/login', component: Login },
  { path: '/register', component: Register },
  { path: '/posts', component: Posts, meta: { requiresAuth: true } },
  { path: '/', redirect: '/posts' }
\]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE\_URL,
  routes
})

router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    if (!localStorage.getItem('token')) {
      next('/login')
    } else {
      next()
    }
  } else {
    next()
  }
})

export default router

### 3.5 Создание компонентов

#### Login.vue

<template>
  <div>
    <h2>Login</h2>
    <form @submit.prevent="login">
      <div>
        <label>Email</label>
        <input v-model="credentials.email" type="email" required>
      </div>
      <div>
        <label>Password</label>
        <input v-model="credentials.password" type="password" required>
      </div>
      <button type="submit">Login</button>
    </form>
    <p>Don't have an account? <router-link to="/register">Register</router-link></p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      credentials: {
        email: '',
        password: ''
      }
    }
  },
  methods: {
    async login() {
      try {
        await this.$store.dispatch('login', this.credentials)
        this.$router.push('/posts')
      } catch (error) {
        alert('Login failed')
      }
    }
  }
}
</script>

#### Register.vue

<template>
  <div>
    <h2>Register</h2>
    <form @submit.prevent="register">
      <div>
        <label>Name</label>
        <input v-model="user.name" required>
      </div>
      <div>
        <label>Email</label>
        <input v-model="user.email" type="email" required>
      </div>
      <div>
        <label>Password</label>
        <input v-model="user.password" type="password" required>
      </div>
      <button type="submit">Register</button>
    </form>
    <p>Already have an account? <router-link to="/login">Login</router-link></p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: '',
        email: '',
        password: ''
      }
    }
  },
  methods: {
    async register() {
      try {
        await this.$store.dispatch('register', this.user)
        this.$router.push('/login')
      } catch (error) {
        alert('Registration failed')
      }
    }
  }
}
</script>

#### Posts.vue

<template>
  <div>
    <h1>Posts</h1>
    <button @click="logout">Logout</button>
    
    <h2>Add New Post</h2>
    <form @submit.prevent="addPost">
      <input v-model="newPost.title" placeholder="Title" required>
      <textarea v-model="newPost.content" placeholder="Content" required></textarea>
      <button type="submit">Add Post</button>
    </form>
    
    <h2>Posts List</h2>
    <div v-if="posts.length === 0">No posts yet</div>
    <div v-else>
      <div v-for="post in posts" :key="post.id" class="post">
        <h3>{{ post.title }}</h3>
        <p>{{ post.content }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      posts: \[\],
      newPost: {
        title: '',
        content: ''
      }
    }
  },
  async created() {
    await this.fetchPosts()
  },
  methods: {
    async fetchPosts() {
      try {
        const response = await axios.get('http://localhost:8000/api/posts', {
          headers: {
            Authorization: \`Bearer ${this.$store.state.token}\`
          }
        })
        this.posts = response.data
      } catch (error) {
        console.error('Error fetching posts:', error)
      }
    },
    async addPost() {
      try {
        await axios.post('http://localhost:8000/api/posts', this.newPost, {
          headers: {
            Authorization: \`Bearer ${this.$store.state.token}\`
          }
        })
        this.newPost = { title: '', content: '' }
        await this.fetchPosts()
      } catch (error) {
        console.error('Error adding post:', error)
      }
    },
    async logout() {
      try {
        await axios.post('http://localhost:8000/api/logout', {}, {
          headers: {
            Authorization: \`Bearer ${this.$store.state.token}\`
          }
        })
        this.$store.commit('LOGOUT')
        this.$router.push('/login')
      } catch (error) {
        console.error('Error logging out:', error)
      }
    }
  }
}
</script>

<style scoped>
.post {
  margin-bottom: 20px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}
</style>

### 3.6 Настройка App.vue

Откройте `src/App.vue` и замените содержимое на:

<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

### 3.7 Настройка main.js

Откройте `src/main.js` и замените содержимое на:

import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import axios from 'axios'

Vue.config.productionTip = false

// Настройка axios
axios.defaults.baseURL = 'http://localhost:8000/api'
axios.interceptors.request.use(config => {
  const token = store.state.token
  if (token) {
    config.headers.Authorization = \`Bearer ${token}\`
  }
  return config
})

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')

### 3.8 Запуск Vue приложения

npm run serve

4\. Проверка работы
-------------------

1.  Откройте браузер и перейдите на `http://localhost:8080`
2.  Вы должны быть перенаправлены на страницу входа
3.  Создайте нового пользователя через страницу регистрации
4.  Войдите в систему
5.  Добавляйте и просматривайте посты

5\. Деплой приложения
---------------------

### 5.1 Бэкенд (Laravel)

1.  Настройте сервер с поддержкой PHP 8.0+
2.  Установите Composer на сервере
3.  Скопируйте файлы Laravel на сервер
4.  Настройте .env файл с реальными данными БД
5.  Выполните:
    
    composer install --optimize-autoloader --no-dev
    php artisan config:cache
    php artisan route:cache
    php artisan view:cache
    php artisan migrate --force
            
    

### 5.2 Фронтенд (Vue.js)

1.  Соберите проект:
    
    npm run build
    
2.  Скопируйте содержимое папки `dist` на хостинг
3.  Настройте базовый URL в `src/main.js`:
    
    axios.defaults.baseURL = 'https://your-domain.com/api'
    

6\. Возможные проблемы и решения
--------------------------------

### 6.1 Ошибки CORS

Убедитесь, что в `config/cors.php` правильно указаны разрешенные домены.

### 6.2 Проблемы с аутентификацией

Проверьте:

*   Действительный токен в localStorage
*   Правильность заголовка Authorization в запросах
*   Работоспособность API через Postman

### 6.3 Ошибки базы данных

Проверьте:

*   Настройки подключения в .env
*   Существование таблиц в БД
*   Права пользователя БД

7\. Дальнейшее развитие
-----------------------

*   Добавление редактирования и удаления постов
*   Пагинация списка постов
*   Категории постов
*   Загрузка файлов
*   Реальные-time обновления через WebSockets
