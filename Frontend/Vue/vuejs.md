# installation
## CDN
```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```
 - At the buttom of body.
# Bits and Pieces
## Vue CLI
 * We select libs we want and it automatically plugs them.
 * Configures webpack for deployment
 * Hot Module Replacement in dev mode.
```bash
npm install -g @vue/cli
vue ctreate project-name
vue ui
```
Add `'plugin:prettier/recommended'` to `'extends'` section in `eslintrc.js`.
### The project structure
 * public
   * Static files that shouldn't go through webpack
   * index.html
     - Webpack will automatically add the correct script tags to the buttom of the body.
       * chunk-vendors file contains all the dependencies
       * app file contains all the application specific code
 * src
   * assets
     * Static files that go through webpack
   * components
   * views
     * Files for different views(pages) of the app
   * App.vue
     * Root component wrapping all other components
   * main.js
     * Renders the app and mounts it to the DOM
   * router.js
     * Vue Router
   * store.js
 * VueX
## Vue Router
## [VueX](vuex.md)
## Utils
### Vue-devtools
Install [vue-devtools](https://github.com/vuejs/vue-devtools#vue-devtools) browser extension/addon
 - If you are using chrome make sure you enable `Allow access to file URLs`
# Core
## Templates
Use backticks(js template literal) for multiline templates.
### Mustache interpolations
```html
<div id="app">
  <p>I have a {{ product }}</p>
  <p>{{ number + 1 }}</p>
  <p>{{ isWorking ? 'YES' : 'NO' }}</p>
  <p v-once>{{ product.getSalePrice() }}</p>
</div>
```
 - `v-once` directive will stop the node from being reactive, affects the expression and
     also every other binding.
 - `v-html` for raw html
   - `{{ rawHtml }}` ⟶ `<span style="color: red">This should be red.</span>`
   - `<span v-html="rawHtml"></span>` ⟶ `<span><span style="color: red">This should be red.</span></span>`
   - Never use `v-html` on untrusted content such as user provided content.
 - Mustache interpolations are sandboxed and only `Math` and `Date` are available, not user
     defined globals

## binding
### Two-way
 - So many form examples in [Vuejs docs](https://vuejs.org/v2/guide/forms.html).
```html
<input v-model="firstName">
       v-model.lazy   ⟶ Syncs input after change event instead of input event.
       v-model.number ⟶ Cast to number
       v-model.trim   ⟶ Strips whitespace
```
 * Works on `Input`, `Textarea`, and `select` elements.
 * Ignores the initial `value`, `checked` or `selected` attributes.
 * It automatically picks the correct way to update the element based on the input type.
 * `v-model` is syntactic sugar, it internally uses different properties and emits 
      different events for eatch input elements type(`Input`, `Textarea`, `select`).

<details><summary>A cool example</summary>
<p>

**html**
```html
<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>
</p>
</details>
```

**js**
```js
new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```

`checkedNames` will be the list of checked names!

<details><summary>Dropdown(select element)</summary>
<p>

**html**
```html
<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<span>Selected: {{ selected }}</span>
```

**js**
```js
new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```
</p>
</details>

<details><summary>Checkbox</summary>
<p>

**html**
```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```

**js**
```js
// when checked:
vm.toggle === 'yes'
// when unchecked:
vm.toggle === 'no'
```
 - browsers don’t include unchecked boxes in form submissions. To guarantee that one of 
     the values is submitted in a form (e.g. “yes” or “no”), use radio inputs instead.
</p>
</details>
</p>
</details>

### One-way
```html
<a v-bind:href="url">...</a> ⟶ <a :href="url">...</a>
 * :disabled="isButtonDisabled”
 * :class="[isActive ? activeClass : '', dataClass, 'strClass', { 'asdf': isAsdf, var: isVar, ... }]
 * :style="{ color: activeColor, fontSize: fontSize + 'px', display: ['-webkit-box', '-ms-flexbox', 'flex'] }"
 * :id="'list-' + id"
 * :[attributeName]="url" 
```
 - If `isButtonDisabled` has the value of `null`, `undefined`, or `false`, the disabled
     attribute will not even be included in the rendered `<button>` element.
 - If `isActive` is `True`, 'active' will be added to the class list.
 - Can have `<a class="static" :class=...>`.
 - class attribute on a custom component(binded or not) will be added to the component’s 
     root element. Existing classes on this element will not be overwritten.
 - The bound object doesn't have to be inline(can be defined in data).
 - In fact its better to always use styleObjects to keep the template clean.
 - Vue will auto-add appropriate vendor prefixes to the applied styles.
 - `attributeName` is a data property of the Vue instance whose value is `'href'`.
 - Dynamic arguments should be either `String` or `null`
 - Dynamic arguments also don't accept space and quotes since they are not accepted 
     inside html attrs. Use computed properties if you can't write your expression 
     wihtout expressions or quotes, also avoid uppercase when using in-DOM templates.
 - Browsers will coerce attr names into lowercase.

## Directives
```html
<template v-if="ok">
  <h1>Title of conditional group</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
<p v-if="inStock">{{ product }}</p>
<p v-else-if="onSale">...</p>
<p v-else>...</p>

<p v-show="showProductDetails">...</p>
```
 * the template tag is an invisible wrapper that won't be rendered.
 * The condition won't change that often ? v-if : v-show
 * Use && for and

```html
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```
 - Switching the loginType won't rerender the elements, it will reuse them and only
     change the placeholder text! So the text we already typed won't change. however, if
     the input tags have unique key attributes they will be rerendered from scratch.
 - v-show doesn't support the `<templage>` element.
## List Rendering
```html
<li v-for="n in 10" :key="n">
    {{n}}
</li>
<cart-product v-for="item in products" :product="item" :key="item.id" />
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```
 * Can be used with a component or with plain HTML
 * Can also use `of` instead of `in` to be closer to js syntax but we don't cuz PYTHON.
 * Vue has an "in-place patch" strategy when updating a list rendered with v-for, to
     change that we should add the unique `:key` attributes. The default has more
     performace but unless you know what you are doing, it's recommended to use `:key`.
 * `:key` is actually requiered in case of componenets.
 * Don't use non-primitive values for key, use string and int.

The instance method `vm.$set` is an alias to the global `Vue.set`.

**Note!**
When used together with `v-if`, `v-for` has a higher priority, so `v-if` will be run on 
each iteration of the loop separately. It's not good style tho.
 - To disable the `v-for` on a condition just use a wrapper or `template` with `v-if`.

### List
```html
    v-for="item in list"
    v-for="(item, index) in list"
```
 * The following mutation methods are available in Vue. Don't use += etc. Only use these
   * push(), pop(), shift(), unshift(), splice(), sort(), reverse()
 * There are also non-mutating methods, e.g. filter(), concat() and slice(), that return
     a new array instead, don't forget to assign the new array to the old one.
   * One would think Vue would throw away old elements and create new ones but view
       reuses the overlapping elements efficiently.
 * **The following are not reactive:**
   * `vm.items[indexOfItem] = newValue // doesn't work`
     * `Vue.set(vm.items, indexOfItem, newValue) // works`
     * `vm.items.splice(indexOfItem, 1, newValue) // works`
   * `vm.items.length = newLength // doesn't work`
     * `vm.items.splice(newLength) // works`

### Object
```html
    v-for="(value, key) in object"
    v-for="(value, name, index) in object"
```
 * Object iteration order is based on `Object.keys()` which is not guaranteed to be
     consistent across JS engine implementations.
 * Vue cannot detect property addition or deletion.
   * `b.a = c // doesn't work`
     * `Vue.set(object, propertyName, value) // works`
If We wanna add multiple new properties we do:
```js
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
Instead of:
```js
Object.assign(vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
To display a filtered or sorted version of an array without mutating the original data:
```js
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
computed: {
  evenNumbers: function () {
    return this.numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```
 * We can't use computed in nested `v-for`. In that case we use a method.

### Example
<details><summary>Simple todo app</summary>
<p>

```html
<div id="todo-list-example">
  <form v-on:submit.prevent="addNewTodo">
    <label for="new-todo">Add a todo</label>
    <input
      v-model="newTodoText"
      id="new-todo"
      placeholder="E.g. Feed the cat"
    >
    <button>Add</button>
  </form>
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:key="todo.id"
      v-bind:title="todo.title"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
```
 * Note the is="todo-item" attr. This is necessary in DOM templates, bc only an `<li>`
     is valid in a `<ul>`. It does the same thing as <todo-item>, but works around a 
     potential browser parsing error.

```js
Vue.component('todo-item', {
  template: `
    <li>
      {{ title }}
      <button v-on:click="$emit('remove')">Remove</button>
    </li>
  `,
  props: ['title']
})

new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [ { id: 1, title: 'Do the dishes', }, ],
    nextTodoId: 2
  },
  methods: {
    addNewTodo: function () {
      this.todos.push({
        id: this.nextTodoId++,
        title: this.newTodoText
      })
      this.newTodoText = ''
    }
  }
})
```

</p>
</details>

## Action/Events
```html
<button v-on:click="addToCart">     ⟶ <button @click="addToCart">
            @click.self="say('hi')" ⟶ only trigger if event.target is element itself
            @click="add(1, $event)" ⟶ 1 and event are passed to add method
<form @submit.prevent="addProduct"> ⟶ prevent default behavior, eg. page reload
             .stop                  ⟶ stop all event propagation
             .capture               ⟶ an event targeting an inner element is handled here before being handled by that element
      @scroll.passive="onScroll"    ⟶ the scroll event's default behavior (scrolling) will happen immediately, instead of waiting for `onScroll` to complete.
<img @mouseover.once="showImage">   ⟶ only trigger once, can also be used on components
     @[eventName]="doSomething"
     @click.[ctrl].[exact]          ⟶ ctrl + anything else whilte click, only ctrl, only click with no modifier pressed
     left, right, middle            ⟶ which mouse click
     @keyup.{ctrl, alt, shift, meta}.c 
     enter, tab, space, delete(del/backspace), esc, left, right, up, down, 
```
 - We might need to infotm the parent component of something.
 - `eventName` is a data property of the Vue instance.
 - Dynamic arguments should be either `String` or `null`
 - Dynamic arguments also don't accept space and quotes since they are not accepted 
     inside html attrs. Use computed properties if you can't write your expression 
     wihtout expressions or quotes, also avoid uppercase when using in-DOM templates.
 - Browsers will coerce attr names into lowercase.
 - `event` will be passed to addToCart.
 - order of chaining matters
   - `:click.prevent.self` will prevent all clicks
   - `:click.self.prevent` will only prevent clicks on the element itself.
 - The .passive modifier is especially useful for improving performance on mobile.
 - Don’t use `.passive` and `.prevent` together, because `.prevent` will be ignored 

**Custom Events**
```html
<button-counter @incrementBy="incWithVal">
```
**Inside parent:**
```js
methods: {
    incWithVal: (toAdd) => { ... }
}
```
**Can use this in child template:**
```js
this.$emit('incrementBy', 5)
```
We can also have:
**child template:**
```html
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```
**parent template:**
```html
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```

**Custom input component that works with v-model:**
**js**
```js
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```

## Computed Properties
```js
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
  fullName_getterOnly: () => {
    returnthis.firstName + " " + this.lastName;
  },
}
```
 * unlike methods, computed properties are cached based on their reactive dependencies.
## Watchers
```js
watch: {
  // whenever question changes, this function will run
  question: function (newQuestion, oldQuestion) {
    ...
  },
  firstName: function (newValue) {
    ...
  },
},
```
## Components
### Layout
```js
import ProductComponent from "./components/ProductComponent"
import ReviewComponent from "./components/ReviewComponent"
Vue.component("my-component", {
  components: {ProductComponent, ReviewComponent},
  props: {
    message: String,
    product: Object,
    email: {
      type: String,
      required: true,
      default: "none",
      validator: (value) => {},
    },
  },
  data: () => {
    return {firstName: "Vue", lastName: "Mastery"};
  },
  computed: {
    fullName: () => {
      returnthis.firstName + " " + this.lastName;
    },
  },
  watch: {
    firstName: (value, oldValue) => { ...  },
  },
  methods: { ... },
  template: "<span>{{ message }}</span>",
});
```
 * The rule of thumb in vue is use never use arrow functions for top-level functions and
 always use arrow functions for everything else.
   * Note that arrow functions in JS don't bind `this`
 * Data should always be function, otherwise all instances of the component will have
     the same data.

### Component Registration
Globally registered components can be used in the template of any root Vue instance (new Vue) created afterwards

### Props

```js
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```
 * Component data are scoped, the inner component doesn't have access to the outer
   component data. To pass data to the child component we should use props.
**html**
```
<product message="hello"></product>
```
**js**
```
Vue.component('product', {
    props: { // We could just set props: [messages,] but of course this is better
        messages: {
            type: String,
            required: true,
            default: "Hi",
        }
    },
})
```
### Data
 * If you define a data somewhere other than in the data option, eg. in created you do
     this.price = 10, that data won't be included in the dependency tree, so if you
     wanna do such a thing you should first define it in data option with some `null`
     value or something and then do whatever you want.
### Slots
 * One might wanna pass actual content to components as in `<p>content</p>`.
 * Slots can contain any template code, including HTML, or even other components:
 * If the template doesn't contain a `<slot>` element, any content provided between its opening and closing tag would be discarded.
 * Everything in the parent template is compiled in parent scope; everything in the child template is compiled in the child scope.
 * For instance in the below example `{{url}}` will be undefined.
**Component template:**
```html
<div class="container">
  <header><slot name="header"></slot></header>
  <main><slot>Default content</slot></main>
  <footer><slot name="footer"></slot></footer>
</div>
```
**Component usage:**
```html
<app-layout url="/home">
    <h1 slot="header">Page title</h1>
    <a href="{{url}}">home</a>
    <p slot="footer">Contact info</p>
</app-layout>
```
there is v-slot and you can pass values to it like v-slot="asdf" or pass own props like
v-slot={ newpropname: oldpropname } or simply {oldname}
### Dynamic components
[A nice example](https://jsfiddle.net/chrisvfritz/b2qj69o1/)
 * This is using component options objects to create anonymous components, could have
     simply used registered component names as well.

</p>
</details>

### DOM Template Parsing Caveats
Some HTML elements, such as `<ul>`, `<ol>`, `<table>` and `<select>` have restrictions
on what elements can appear inside them, and some elements such as `<li>`, `<tr>`, and 
`<option>` can only appear inside certain other elements. So we use the is attribute.
```html
<table>
  <tr is="blog-post-row"></tr>
</table>
```
 * this limitation does not apply if you are using a string template of the forms:
   *  String templates (e.g. template: '...')
   *  Single-file (.vue) components
   *  `<script type="text/x-template">`

## Lifecycle Hooks
![](https://vuejs.org/images/lifecycle.png)

 * beforeCreate, created 
 * beforeMount, mounted
     * To put code that should be mounted as soon as the component is mounted to the DOM
    ```js
    mounted() {
        eventBus.$on('review-submitted', productReview => {
            this.reviews.push(productReview)
        })
    }
    ```
 * beforeUpdate, updated
 * beforeDestroy, destroyed

## Filters
Like django filters, can be used in mustache interpolations and v-bind expressions
**local filters with component options**
```js
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}
```
**global filters before Vue instantiation**
```js
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})
```
 * In case of conflict, local is prioritized
 * Can be chained and can have extra args: `{{ message | filterA('arg1', arg2) }}`


## Mixins
Mixins are a flexible way to distribute reusable functionalities for Vue components. 
A mixin object can contain any component options. When a component uses a mixin, all 
options in the mixin will be “mixed” into the component’s own options.

```js
// define a mixin object
var myMixin = {
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  }
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// define a component that uses this mixin
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "hello from mixin!"

new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye',
      bar: 'def'
    }
  },
  created: function () {
    console.log(this.$data)
    // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})
```
 * In case of overlapping options, they will be “merged” using appropriate strategies.
 * Data objects merge recursively, with the component’s data taking priority.
 * Hooks are merged into an array so that all of them will be called. Mixins run first.

# Reactivity
![](https://d2mxuefqeaa7sj.cloudfront.net/s_95C7E286F3389C1D370227B8CED4CF94CF293E39D5CCC0F505B76EEB1A0EA8FE_1524258939508_image.png)
 * `Date.now()` is not a reactive dependency.
   * You should use a method if you want it to update.


# Patterns

<details><summary>Smarter Watches</summary>
<p>

**bad**
```js
created() {
    this.fetchUserList()
},
watch: {
    searchText() {
        this.fetchUserList()
    }
}
```
**better**
```js
created() {
    this.fetchUserList()
},
watch: {
    searchText: 'fetchUserList'
}
```
**best**
```js
watch: {
    searchText: {
        handler: 'fetchUserList', // can use a function too
        immediate: true
    }
}
```

</p>
</details>

<details><summary>Component Registration</summary>
<p>

**bad**
**js:**
```js
import BaseButton from './base-button'
import BaseIcon from './base-icon'
import BaseInput from './base-input'

export default {  
    components: {    
        BaseButton,    
        BaseIcon,    
        BaseInput
    }
}
```
**html:**
```html
<BaseInput 
  v-model="searchText" 
  @keydown.enter="search"
/>
<BaseButton @click="search">
  <BaseIcon name="search"/>
</BaseButton>
```
**best**
```js
import Vue from 'vue' 
import upperFirst from 'lodash/upperFirst' 
import camelCase from 'lodash/camelCase' 
// Require in a base component context 
const require Component = require.context( // Require.context is a webpack feature
    './components', false, /base-[\w-]+\.vue$/ 
    // directory             filename regex
) 

requireComponent.keys().forEach(fileName => { 
    // Get component config 
    const componentConfig = requireComponent(fileName)

    // Get PascalCase name of component
    const componentName = upperFirst(
        camelCase(fileName.replace(/^\.\//, '').replace(/\.\w+$/, ''))
    )

    // Register component globally
    Vue.component(componentName, componentConfig.default || componentConfig)
    //                           when you exportdefault     if you don't export default
    //                           that obj is added to a     eg. if you only have a
    //                           default property which     template in your component
    //                           is automatically handled   then the options would be
    //                           with import.               just the componentConfig
})
```
 * Can have this code in source, main.js, the entry file for the app.
 * It should be before instantiating Vue, could also put in a _globals file in
     components directory and then import it into entry file.
 * Don't need all that JS anymore for such a simple template.

</p>
</details>

<details><summary>Module Registration</summary>
<p>

**bad**
```js
import auth from './modules/auth' 
import posts from './modules/posts' 
import comments from './modules/comments' 
// ... 

export default new Vuex.Store({ 
    modules: { 
        auth, 
        posts, 
        comments, 
        // ... 
    } 
})
```

**better**
```js
import camelCase from 'lodash/camelCase' 
const requireModule = require.context('.', false, /\.js$/) 
const modules = {} 

requireModule.keys().forEach(fileName => { 
    // Don't register this file as a Vuex module 
    if (fileName === './index.js') return 

    const moduleName = camelCase( 
        fileName.replace(/(\.\/|\.js)/g, '') 
    ) 
    modules[moduleName] = requireModule(fileName) 
}) 

export default modules
```

**best**
```js
modules[moduleName] = { 
    namespaced: true, 
    ...requireModule(fileName), 
}
```
 * index.js file inside modules directory.
So the code becomes:
```js
import modules from './modules' 

export default new Vuex.Store({ 
    modules 
})
```

</p>
</details>

<details><summary>Lazy-Loaded Routes</summary>
<p>

**bad**
```js
{ 
    path: '/admin', 
    name: 'admin-dashboard', 
    component: require('@views/admin').default 
}
```

**best**
```js
{ 
    path: '/admin', 
    name: 'admin-dashboard', 
    component: () => import('@views/admin')
}
```
 * Only 3% of users are admins, so it's not good to have all users download the admin
     dashboard in case they are admins.
 * In webpack you can replace a component with a function that returns a dynamic import.
 * Now only when a user goes to that route they will download the code for this
     component and it's subs that are only used in the admin dashboard.
 * We don't want to do this with all components tho. because then the whole app will
     feel slowish all the time. And also such little splits aren't really saving much.

</p>
</details>

<details><summary>Transparent Wrappers</summary>
<p>

**bad**
BaseInput template:
```html
<template> 
  <label> 
    {{ label }} 
    <input 
      :value="value" 
      @input="$emit('input', $event.target.value)" // so it would work with v-model
    > 
  </label> 
</template>
```
BaseInput usage:
```html
<BaseInput @focus.native="doSomething">
```
 * It would have worked if we didn't have the label because template is transparent.
 * But with label it won't work as we expect because the code would listen to the focus
     event of the root element(label), not the input.

**better**
```html
<template> 
  <label> 
    {{ label }} 
    <input 
      :value="value" 
      v-on="listeners"
    > 
  </label> 
</template>
```

```js
computed: { 
  listeners() { 
    return { 
      ...this.$listeners, 
      input: event => 
        this.$emit('input', event.target.value) 
    } 
  } 
}
```
 * $listeners is an object of all listeners passed to the component from the parent.
 * Overriding input so it works with v-model.
 * We can turn `@focus.native` to `@focus`.
 * Still, if we add an attr to the component that is not defined as a prop it will be
     added to the root element of the template. So for example if we add placeholder
     which does a think in input element it would be added to the label element and so
     do nothing!
 * So we add `inheritAttrs: false` to the component options and we also add
     `v-bind="$attrs"` to the `<input>`. 
 * `$attrs` is an object that contains all the attrs that are passed to the component
     and are not defined as props.
 * Now `BaseInput` would work exactly like an `<input>`.
        
**best**
```js
code
```

</p>
</details>

<details><summary>Single-Root Components</summary>
<p>

**bad**
```html
NavBar template:
<template> 
  <ul> 
    <NavBarRoutes :routes="persistentNavRoutes"/> 
    <NavBarRoutes 
      v-if="loggedIn" 
      :routes="loggedInNavRoutes" 
    /> 
    <NavBarRoutes 
      v-else 
      :routes="loggedOutNavRoutes" 
    />
  </ul> 
</template>
```
NavBarRoutes template:
```html
<template> 
  <li 
    v-for="route in routes" 
    :key="route.name" 
  > 
    <router-link :to="route"> 
      {{ route.title }} 
    </router-link> 
  </li> 
</template>
```
 * So we get single-root component error, if we wrap in a `div` we will get invalid html.

hack:
```js
functional: true, 
render(h, { props }) { 
  return props.routes.map(route =>
    <li :key={route.name}> 
      <router-link :to={route}> 
        {route.title} 
      </router-link> 
    </li> 
  )
}
```
 * A functional component, when using a render function, can actually return an array of
     elements. Because a functional component will actually render in it's parents
     context, and the parent only returns one root element.

</p>
</details>

<details><summary>Factorizing v-if</summary>
<p>

```js
functional: true,
render (h, { props }) {
  return props.show && [
    <TheaterCurtain direction='center'/>
    <TheaterCurtain direction='center'/>
    <TheaterCurtain direction='center'/>
  ]
}
```
 * The trick is that we can return multiple root elements in functional components.

</p>
</details>

[More patterns](https://github.com/chrisvfritz/vue-enterprise-boilerplate)

# Userful Resources:
[awesome-vue](https://github.com/vuejs/awesome-vue/blob/master/README.md)
