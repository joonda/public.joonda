---
title: vue-router에 대해서 파고들기
description: deep-dive about vue-router
tags:
  - Vue
date: 2025-04-10
draft: false
---
## Vue.js를 공부하는 이유
현재 회사에서 내가 들어오기 전, ERP 프로젝트를 진행하고 있었다. 내가 합류했을 당시에는 Thymeleaf + Spring 조합으로 진행하는 걸로 이야기가 됐었지만,
UI에 대한 이슈가 너무 컸었다.

제대로 된 화면이 아닌 급조된 화면에 티가 났었고, 무엇보다 ERP 특성상 반복되는 UI가 많기 때문에 재사용이 용이한 프레임워크가 필요하다고 판단하였다.
물론 React 라는 선택지가 있었지만, 러닝커브가 낮고 빠르게 배워서 접목할 수 있는 프레임워크로 Vue.js가 선택이 되었고 현재 시간을 따로 내면서 공부를 진행하고 있다.

나름 풀스택에 대한 로망도 있었고 프론트엔드 프레임워크도 하나쯤 배워두는것도 좋은 경험이 될 것 같아 흔쾌히 진행하겠다고 했다 (강의 제공 등 편의를 많이 봐주신 것도 한 몫 했다.)

## vue-router가 무엇일까?
vue-router는 Vue에서 제공하는 SPA를 만들기 위한 공식 라우팅 라이브러리이다.
일반적으로 페이지 이동 시, 전체 페이지를 새로고침하지만, Vue에서는 컴포넌트 단위로 화면만 바뀌도록 구성할 수 있다.

## 사용법

### 0. terminal
```terminal
npm install vue-router
```

### 1. `router` > `index.js`
```js
import { createRouter, createWebHistory } from 'src/post/vue/vue-router.mdx'
import AboutView from '@/views/AboutView.vue'
import HomeView from '@/views/HomeView.vue'

const routes = [
  {path: '/', component: HomeView, name: 'Home'},
  {path: '/about', component: AboutView, name: 'About'},
]

const router = createRouter({
  history: createWebHistory('/'),
  routes,
})

export default router
```
* `routes` 배열에 각 경로와 연결한 컴포넌트를 지정한다.
* `createRouter`를 사용하여 라우터 인스턴스 생성, `History` 모드와 이전에 선언했던 `routes`를 설정한다.

### 2. `main.js`
```js
import 'bootstrap/dist/css/bootstrap.min.css'
import 'bootstrap-icons/font/bootstrap-icons.css'

import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
import 'bootstrap/dist/js/bootstrap.bundle.js'
```
* `use(router)` 를 등록한다

### 3. `layouts` > `AppView.vue`
```js
<template>
  <div class="container py-4">
    <RouterView></RouterView>
  </div>
</template>

<script>
export default {}
</script>

<style scoped></style>
```
* `<RouterView>` 컴포넌트를 활용하여 경로에 따라 알맞은 페이지 컴포넌트를 렌더링 한다.

### 4. `layouts` > `AppHeader.vue`
```javascript
<template>
  <header>
    <nav class="navbar navbar-expand-sm navbar-dark bg-primary">
      <div class="container-fluid">
        // ... 생략
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav">
            <li class="nav-item">
              <RouterLink class="nav-link active" to="/">Home</RouterLink>
            </li>
            <li class="nav-item">
              <RouterLink class="nav-link active" to="/about">About</RouterLink>
            </li>
          </ul>
        </div>
      </div>
    </nav>
  </header>
</template>

<script>
export default {}
</script>

<style scoped></style>
```
* `<RouterLink>` 컴포넌트를 활용, `to`옵션으로 경로 지정을 해준다.
* 이를 통해 페이지 새로고침 없이 `Vue Router`가 내부적으로 경로를 전환

## 추가
### 1. `useRouter`
```javascript
<template>
  <div>
    <h2>Home View</h2>
    <button class="btn btn-primary" @click="goAboutPage">About으로 이동</button>
  </div>
</template>

<script setup>
import { useRouter } from 'src/post/vue/vue-router.mdx'
const router = useRouter()
const goAboutPage = () => {
    router.push('/about')
}
</script>
```
* `@click`에 커스텀 메서드인 `goAboutPage` 메서드를 지정, `router.push` 메서드로 구현

### 2. 동적 경로
* `router` > `index.js`
```javascript
// ... 생략

const routes = [
    // ... 생략
  {
    path: '/posts/:id',
    name: 'PostDetail',
    component: PostDetailView,
  },
  {
    path: '/posts/:id/edit',
    name: 'PostEdit',
    component: PostEditView,
  },
]
// ... 생략
```
* `:id` 같은 경우, 해당 경로에 어떤 문자가 오든 상관 없다.
* `$route`의 내장 메서드를 활용하여 해당 `parameter`를 받을 수 있다.
    * `$route.params` > `/post/alice`로 요청 시, `{"id", "alice"}` 출력
    * `$route.query` > `/posts/alice?search=vue3`로 요청 시, `{"search", vue3}` 출력
    * `$route.hash` > `/posts/alice#hashvalue`로 요청 시, `#hashvalue` 출력

### 3. 404 Not Found
* `:id`처럼 일반 파라미터는 URL 경로 중 슬래시(`/`)로 구분된 하나의 세그먼트만 매칭된다.
    * `/user/123` > `:id`는 `123`과 매칭
    * `/user/123/abc` > `:id`는 `123` 까지만 매칭되고 이후는 별도의 `route` 필요
* 정규식을 활용한 파라미터는 `:paramName(정규식)` 형태로 사용되며, 더 유연한 매핑이 가능하다.
```javascript
import NotFoundView from '@/views/NotFoundView.vue'
const routes = [
    // ... 생략
  {
    path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFoundView
  }
]
```
* `/:pathMatch(.*)*`는 어떤 경로든 모두 매칭되며, 주로 `404 Not Found`처리에 사용
    * 여기서 `(.*)`는 모든 문자열을 의미하며, `*`를 붙여 여러 세그먼트까지 포함

### 4. 중첩 라우트
* 특정 페이지 안에 `RouterView`를 중첩하여 설정할 수 있다.
* `router` > `index.js`
```javascript
import NotFoundView from '@/views/NotFoundView.vue'
const routes = [
    // ... 생략
  {
    path: '/nested',
    name: 'Nested',
    component: NestedView,
    children: [
      {
        path: '',
        name: 'NestedHome',
        component: NestedHomeView,
      },
      {
        path: 'one',
        name: 'NestedOne',
        component: NestedOneView,
      },
      {
        path: 'two',
        name: 'NestedTwo',
        component: NestedTwoView,
      },
    ],
  },
]
```
* `routes`에서 `children` 메서드로 배열을 지정하여 중첩 라우트를 적용할 수 있다.
* `path`의 경우 `''`는 `/nested`와 동일, `one`은 `/nested/one`으로 지정된다.
