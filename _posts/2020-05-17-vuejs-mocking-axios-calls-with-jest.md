---
title: "Vue.js - Mocking Axios calls with Jest"
date: 2020-05-20 14:00:00 +0800
categories: [Vue.js]
tags: [programming, tutorial, vuejs, unit test, axios]
---

Create unit tests for your code is essential. It helps you avoid code regression and help others quickly understand what a component is supposed to do.
If your code is correctly tested and continuous integration is configured on your code repository, you are 100% sure that what is committed to the repository never breaks what already exists.

Unfortunately, correctly test your code can be a little tricky, especially if you are using external API as you cannot use them during your tests, which means you need to mock those calls to manage by yourself what you receive.

In this blog post, we review how we can test code, which makes API calls and then creates Vue components based on the result of the request. The API calls are performed with _axios_, and we use the framework _Jest_  for our unit tests.

If you already have a Vue.js project, you can directly jump to the “Write the tests” section to find how writes the tests.

_Sources for this post are available on [GitHub](https://github.com/Patmol/vue-api-tests "GitHub")_

## Create and configure the Vue.js application
Let’s start by creating a simple Vue.js application with the following command
vue create vue-tests
During the creation process, choose the option **Manually select features** and select **Babel** and **Unit Testing**.
Then, when you need to choose the unit testing solution, select **Jest**.
For the other options, you can select each time the option you preferred.

Once the creation is finished, go to the folder of the application with the command `cd vue-tests`.
You can also open the project in your favorite editor and remove the following files :
* src/components/HelloWorld.vue
* tests/unit/example.spec.js

## Create API services
To make our API calls, you need _axios_, which you can install with this npm command `npm i axios`.
Then, in the project, under the **src** folder, you can create a folder ** services** with a JavaScript file inside ** post-service.js**.

 This service contains a single method to retrieve JSON data from [jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com "jsonplaceholder.typicode.com").

```js
import axios from 'axios';

export default {
  getPosts: async () => axios.get('https://jsonplaceholder.typicode.com/posts'),
};
```

## Create models
Once the service has been coded,  you can create a new folder **models**, still under **src**, with a JavaScript file inside named ** post.js** and with this content
```js
export default class Post {
  constructor(id, title, body) {
    this.id = id;
    this.title = title;
    this.body = body;
  }
}
```

## Work on the Vue.js components
Now that you have both your service and your model, you can create two Single File Components in the **components** folder
* Post.vue
* Posts.vue
Those components are very simple as the goal here is not to show how Vue.js components are working but to test them.

### Post.vue
The **Post.vue** component displays a single blog post, with just a title and a body.

```js
<template>
  <article>
      <h2>
          {{ post.title }}
      </h2>
      <p>
          {{ post.body }}
      </p>
  </article>
</template>

<script>
import Post from '@/models/post';

export default {
  name: 'PostComponent',
  props: {
    post: Object,
  },
};
</script>

<style>

</style> 
```

### Posts.vue
The goal of the **Posts.vue** component is to display all posts loaded through the service created earlier.
```js
<template>
  <div class="align-left">
      <post-component v-for="post in posts" :post="post" :key="post.id"></post-component>
      <span class="error" v-if="error">{{ error }}</span>
  </div>
</template>

<script>
import PostComponent from '@/components/Post.vue';
import postService from '@/services/post-service';

export default {
  name: 'PostsComponent',
  components: {
    PostComponent,
  },
  data() {
    return {
      posts: [],
      error: null,
    };
  },
  async created() {
    this.error = null;

    try {
      const response = await postService.getPosts();
      this.posts = response.data;
    } catch (error) {
      this.error = error;
    }
  },
};
</script>

<style scope>
.align-left {
    text-align: left;
}
</style>
```

### App.vue
Before writing the tests, we need to update the **App.vue** to display the lists of posts on the website.
```js
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <posts-component></posts-component>
  </div>
</template>

<script>
import PostsComponent from '@/components/Posts.vue';

export default {
  name: 'App',
  components: {
    PostsComponent,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```
Once all those changes has been done, you can run the Vue.js application to check if everything run correctly or not with the command `npm run serve`.

If everything is right, you must see a page with a lot of _h2_ titles followed each time by a _p_ content.

## Write the tests
Now that we have the structure of our project, we can finally start to write the tests.

### Create mock data for our API service.
Let’s start by creating a JSON file to store the data our API is sending back. We could have put those data directly in our test files, but it’s cleaner to put them in separate files.
The data are stored in a JSON files store in a **data** folder, create under **test**. The JSON file stores two posts we are sending back from our mock API.
The content of `posts-2.josn`
```json
[{
    "userId": 1,
    "id": 1,
    "title": "Hello World 1",
    "body": "Hello World 1 Body"
},
{
    "userId": 1,
    "id": 2,
    "title": "Hello World 2",
    "body": "Hello World 2 Body"
}]
```

### Testing the PostComponent
Firstly, we test our PostComponent. This component does not call an API, but it allows us to introduce some Vue.js test elements.
In the folder **test/unit**, create a file name ** post.spec.js**. 

In this file, we start by importing various elements
```js
import { shallowMount } from '@vue/test-utils';
import PostComponent from '@/components/Post.vue';

import Post from '@/models/post';
```

- `shallowMount` returns a Vue component with some helpers to interact with it. The advantage of `shallowMount` over `mount` is that `shallowMount` not render child component. Which means we test our component in isolation, and if the component has a lot of children, we don’t overload our tests by creating all those children.
- `PostComponent` is the component we are testing
- `Post` is the model used to pass data to the component

Next, we can create a block of tests with `describe` and a test in it with `it`.
```js
describe('Post.vue', () => {
  it('display the title and body of a post', () => {
  });
});
```

In the test itself, we create the data of the test
```js
const post = new Post(1, 'Hello World', 'Hello World Body');
```

And then, an instance of our Post component with filled properties.
```js
const wrapper = shallowMount(PostComponent, {
	propsData: { post },
});
```

Finally, we need to test if our component work like expected by checking the content of the `h2` and of the `p`.
```js
// We check the content of the title
expect(wrapper.get('h2').text()).toEqual(post.title);
// We check the content of the body
expect(wrapper.get('p').text()).toEqual(post.body);
```

_You can find this file on [GitHub](https://github.com/Patmol/vue-api-tests/blob/master/tests/unit/post.spec.js "GitHub - post.spec.js")._

### Testing our PostsComponent
Now we have tested a simple Vue component; we need to check our component with the API call, it’s kind the goal since the beginning of the post.

Wait no more and create a **posts.spec.js** file in the **unit** folder.

As for the previous test, we need to import various elements in our test
```js
import { shallowMount } from '@vue/test-utils';
import axios from 'axios';
import PostsComponent from '@/components/Posts.vue';
import PostComponent from '@/components/Post.vue';
import data2 from '../data/posts-2.json';
```

- `axios` is used for our API calls
- `PostsComponent` is the component we are testing
- `PostComponent` is a child component
- `data2` are the data send back by the API

And now, the fun is beginning ! We need to mock, with jest, the `get` method from `axios` and to perform this, under the `import`s we add the following code 
```js
jest.mock('axios', () => ({
  get: jest.fn(),
}));
```

Then, before each test, we need to clear the mock and reset the return value of the get method.
```js
beforeEach(() => {
  axios.get.mockClear();
  axios.get.mockReturnValue(Promise.resolve({}));
});
```
This code can be added just after the `jest.mock()` one.

Now that everything is set up, we can write our test by creating a block of test and the test itself
```js
describe('Posts.vue', () => {
  it('shoud display the 2 loaded posts', async () => {

  });
});
```

In the test, we start by setting the return of the get method from axios this test (as in the `beforeEach`, it to reset it)
```js
axios.get.mockReturnValue(Promise.resolve({
	data: data2,
}));  
```

This code is saying “when the get method of axios is called, return a Promise which returns an object with a property data, and this property data is the content of the `data/posts-2.json` file (thanks to the `import`).

Next, we shallowMount our component
```js
const wrapper = shallowMount(PostsComponent);
```

And by performing this action, the call to the API is supposed to be performed.
We can test if it’s the case
```js
// We need to wait for a first tick for the call to the API
await wrapper.vm.$nextTick();
expect(axios.get).toHaveBeenCalledTimes(1);
```

The call is performed, but we need to test if the result of the call is correctly used. To do this, we check if two **PostComponent** has been created or not as children of our component
```js
// We need to wait for a second tick for the refresh of the view
await wrapper.vm.$nextTick();
// We check if we have created one child by data received and that those children are PostComponent.
expect(wrapper.vm.$children.length).toEqual(2);
```

And we also check if those children are PostComponent or not
```js
wrapper.vm.$children.forEach((child) => {
	expect(child.$options.name.toString()).toEqual(PostComponent.name);
});
```

And it’s finished! We have tested our component!
Easy, no? 
_You can find this file on [GitHub](https://github.com/Patmol/vue-api-tests/blob/master/tests/unit/posts.spec.js "GitHub - post.spec.js"), there are more tests on GitHub, but the idea behind them is always the same__

## Testing components where API calls are performed
Testing those components was quite easy. We just need to mock the method from `axios` used by our components and simulate a return of the method with the wanted object.

Now you know of to do it; you don’t have any excuse anymore not to test them :-)