# vue-kindergarten

[![Build Status](https://travis-ci.org/JiriChara/vue-kindergarten.svg?branch=master)](https://travis-ci.org/JiriChara/vue-kindergarten)
[![Test Coverage](https://lima.codeclimate.com/github/JiriChara/vue-kindergarten/badges/coverage.svg)](https://lima.codeclimate.com/github/JiriChara/vue-kindergarten/coverage)
[![NPM Version](https://img.shields.io/npm/v/vue-kindergarten.svg)](https://www.npmjs.com/package/vue-kindergarten)
[![NPM Dowloads](https://img.shields.io/npm/dm/vue-kindergarten.svg)](https://www.npmjs.com/package/vue-kindergarten)
[![Dependency Status](https://gemnasium.com/badges/github.com/JiriChara/vue-kindergarten.svg)](https://gemnasium.com/github.com/JiriChara/vue-kindergarten)

## Introduction

**vue-kindergarten** is a plugin for VueJS 2.0 that integrates [kindergarten](https://github.com/JiriChara/kindergarten) into your VueJS applications. It helps you to authorize your components, routes and the rest of your application in very modular way. If you are not familiar with **kindergarten** yet, I highly recommend you to check out [the README](https://github.com/JiriChara/kindergarten) first.

## Installation

```
yarn add vue-kindergarten
# or
npm install vue-kindergarten
```

And you can register the plugin like this:

```js
import Vue from 'vue';
import VueKindergarten from 'vue-kindergarten';

import App from './App';
import router from './router';
import store from './store';

Vue.use(VueKindergarten, {
  // Getter of your current user.
  // If you use vuex, then store will be passed
  child: (store) => {
    return store.state.user;
    // or
    // return decode(localStorage.getItem('jwt'));
    // or your very own logic..
  }
});

new Vue({
  el: '#app',
  router,
  store,
  template: '<App/>',
  components: { App },
});
```

## Usage

First we need to define our perimeters. Perimeter is a module that represents some part of your applications or a business domain. It defines rules that has to be respected and can additionally expose some methods that you can use in your application.

```js
import { createPerimeter } from 'vue-kindergarten';

createPerimeter({
  purpose: 'article',

  can: {
    read: () => true

    // only admin or moderator can update articles
    update(article) {
      return this.isAmin() || (this.isCreator(article) && this.isModerator());
    },

    // if user can update articles then she can also destroy them
    destroy(article) {
      return this.isAllowed('update', article);
    }
  },

  secretNotes(article) {
    this.guard('update', article);

    return article.secretNotes;
  },

  isAdmin() {
    return this.child.role === 'admin';
  },

  isModerator() {
    return this.child.role === 'moderator';
  },

  isCreator(article) {
    return this.child.id === article.author.id;
  },

  expose: [
    'secretNotes'
  ]
});
```

```vue
<template>
  <main>
    <article v-for="article in articles.items" v-show="$isAllowed('read')">
      <h1>{{ article.title }}</h1>

      <router-link :to="`/article/${article.id}/edit`" v-show="$article.isAllowed('update', article)">
        Edit Article
      </router-link>

      <p>{{ article.content }}</p>

      <p>{{ $article.secretNotes() }}</p>
    </article>
  </main>
</template>

<script>
  import { mapState } from 'vuex';

  export default {
    computed: {
      ...mapState([
        'articles'
      ])
    },

    // add your perimeters
    perimeters: [
      articlesPerimeter
    ]
  }
</script>
```

In example above we have injected our `articlesPerimeter` into our component. Our component act as sandbox now. We can call all the methods that are available in the Sandbox directly on our component.

### Protecting Routes

```js
import Router from 'vue-router';

import Home from '@/components/Home';
import Articles from '@/components/Articles';
import EditArticle from '@/components/EditArticle';
import RouteGoverness from '@/governesses/RouteGoverness';

const router = new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home
    },

    {
      path: '/articles',
      name: 'articles',
      component: Articles,
      meta: {
        perimeter: articlesPerimeter,
        perimeterAction: 'read',
      }
    },

    {
      path: '/articles/:id/edit',
      name: 'edit-article',
      component: EditArticle,
      meta: {
        perimeter: articlesPerimeter,
        perimeterAction: 'update',
      }
    }
  ]
});

router.beforeEach((to, from, next) => {
  to.matched.some((routeRecord) => {
    const perimeter = routeRecord.meta.perimeter;
    const Governess = routeRecord.meta.governess || RouteGoverness;
    const action = routeRecord.meta.perimeterAction || 'route';

    if (perimeter) {
      const sandbox = createSandbox(child(), {
        governess: new Governess({
          from,
          to,
          next,
        }),

        perimeters: [
          perimeter,
        ],
      });

      return sandbox.guard(action);
    }

    return next();
  });
});

export default router;
```

#### Route Governess

```js
import { HeadGoverness } from 'vue-kindergarten';

export default class RouteGoverness extends HeadGoverness {
  guard(action, { next }) {
    // or your very own logic to redirect user
    // see. https://github.com/JiriChara/vue-kindergarten/issues/5 for inspiration
    return this.isAllowed(action) ? next() : next('/');
  }
}
```

## More About Vue-Kindergarten

[Role Based Authorization for your Vue.js and Nuxt.js Applications Using vue-kindergarten](https://medium.com/@JiriChara/role-based-authorization-for-your-vue-js-and-nuxt-js-applications-using-vue-kindergarten-fd483e013ec5#.y0xnnidl6)

## License

The MIT License (MIT) - See file 'LICENSE' in this project

## Copyright

Copyright © 2017 Jiří Chára. All Rights Reserved.
