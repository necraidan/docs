---
title: Support de TypeScript
description: Nuxt.js est livré avec le module @nuxtjs/typescript avec le support natif de TypeScript.
---

> Un système de typage statique peut aider à prévenir de nombreuses erreurs potentielles à l'exécution, en particulier lorsque les applications grandissent.
>
> C'est pourquoi le tout nouveau package `@nuxt/typescript` de Nuxt est livré avec le support natif des outils TypeScript
> - Les définitions de type officiels Nuxt
> - Autocompletion dans l'IDE
> - Tout écrire en TypeScript (`layouts`, `pages`, `components`, `plugins`, `store`)
> - Support TS à l'éxécution (`nuxt.config.ts`, `modules`, `serverMiddlewares`)
> - Support TSX.

## Commencer

Pour utiliser TypeScript dans votre projet, installer http://`@nuxt/typescript` dans `devDependencies` et `ts-node` dans `dependencies`:

```sh
npm i -D @nuxt/typescript
npm i ts-node
# OU
yarn add -D @nuxt/typescript
yarn add ts-node
```

<div class="Alert Alert--gray">

**INFO:** `@nuxt/typescript` fournit les dépendances liée à TypeScript nécessaires à la compilation des fichiers TypeScript et des types de contrôle dans un processus séparé.

</div>

<div class="Alert Alert--gray">

**INFO:** `ts-node` extends `@nuxt/core` to enable runtime TypeScript support for `nuxt.config.ts` & `serverMiddlewares`.

</div>

You'll also need to create an empty `tsconfig.json` file in your root project folder, through either a code editor or command line:

```sh
touch tsconfig.json
```

<div class="Alert Alert--gray">

**INFO:** The `tsconfig.json` file will automatically update with default values the first time you run the `nuxt` command.

</div>

## From JavaScript to TypeScript

### Configuration file

To use TypeScript in your configuration file, all you need to do is rename `nuxt.config.js` in your root project folder to `nuxt.config.ts`.

Nuxt.js also brings type definitions which provides autocompletion and type checking:

```ts
import NuxtConfiguration from '@nuxt/config'

const config: NuxtConfiguration = {
  // Type or Press `Ctrl + Space` for autocompletion
}

export default config
```

### Components

For components, we highly advise you to use [vue-property-decorator](https://github.com/kaorun343/vue-property-decorator) which depends on [vue-class-component](https://github.com/vuejs/vue-class-component).

Here is a basic example of mixing a `page` with a reusable `component` to display data fetched with Nuxt's [`asyncData`](https://nuxtjs.org/guide/async-data) method.

```ts
/* models/Post.ts */

export default interface Post {
  id: number
  title: string
  description: string
}
```

```html
<!-- components/PostPreview.vue -->

<template>
  <div>
    <h2>{{ post.title }}</h2>
    <p>{{ post.description }}</p>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'
import Post from '~/models/Post'

@Component
export default class PostPreview extends Vue {
  @Prop({ type: Object, required: true }) post!: Post
}
</script>
```

```html
<!-- pages/feed.vue -->

<template>
  <div>
    <PostPreview v-for="post in posts" :key="post.id" :post="post" />
  </div>
</template>

<script lang="ts">
import axios from 'axios'
import { Component, Vue } from 'vue-property-decorator'
import Post from '~/models/Post'

@Component({
  components: {
    PostPreview: () => import('~/components/PostPreview.vue')
  },
  async asyncData () {
    let { data } = await axios.get(`https://my-api/posts`)
    return {
      posts: data
    }
  }
})

export default class FeedPage extends Vue {
  posts: Post[] = []
}
</script>
```

You can use the exact same logic for `layouts`.

## Linting with ESLint

If you're using ESLint to lint your project, here is how you can make ESLint lint your TypeScript files.

<div class="Alert Alert--teal">

**IMPORTANT:** We're assuming you have already set up [nuxt/eslint-config](https://github.com/nuxt/eslint-config) within your project.

</div>

First, you need to install @typescript-eslint/eslint-plugin and @typescript-eslint/parser ([GitHub monorepo for typescript-eslint](https://github.com/typescript-eslint/typescript-eslint)):

```sh
npm i -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
# OR
yarn add -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

Then, edit your ESLint configuration `.eslintrc.js` by adding the `@typescript-eslint` plugin and making `@typescript-eslint/parser` the default parser.

A minimal configuration should look like this:

```js
module.exports = {
  plugins: ['@typescript-eslint'],
  parserOptions: {
    parser: '@typescript-eslint/parser'
  },
  extends: [
    '@nuxtjs'
  ],
  rules: {
    '@typescript-eslint/no-unused-vars': 'error'
  }
}
```

Finally, add or edit the `lint` script of your `package.json`:

```json
"lint": "eslint --ext .ts,.js,.vue --ignore-path .gitignore ."
```

<div class="Alert Alert--gray">

**INFO:** The `--ignore-path` option is useful in preventing ESLint from linting files/folders like `node_modules`, `.nuxt` or any others you don't want to lint.

</div>

You can now lint your TypeScript files by running `npm run lint` (or `yarn lint`).
