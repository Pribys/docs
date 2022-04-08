In addition, `type` can also be a custom class or constructor function and the assertion will be made with an `instanceof` check. For example, given the following class:

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }
}
```

You could use it as a prop's type:

<div class="composition-api">

```js
defineProps({
  author: Person
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    author: Person
  }
}
```

</div>

to validate that the value of the `author` prop was created with `new Person`.

## Boolean Casting

Props with `Boolean` type has special casting rules to mimic the behavior of native boolean attributes. Given a `<MyComponent>` with the following declaration:

<div class="composition-api">

```js
defineProps({
  disabled: Boolean
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    disabled: Boolean
  }
