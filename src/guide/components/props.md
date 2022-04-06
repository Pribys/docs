# Props

> Se supone que ya has leído [Components Basics](/guide/essentials/component-basics). Si no es así, y eres novato, se recomienda empezar por ahí.

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/vue-3-reusable-components-with-props" title="Free Vue.js Props Lesson"/>
</div>

## Declaración de Props

Los componentes de Vue requieren una declaración explícita de props, para que Vue sepa qué elementos externos pasados al componente deben tratarse como atributos fallidos (que se analizarán en la siguiente sección).

<div class="composition-api">

En los CAU que usan `<script setup>`, los props se pueden declarar usando la macro `defineProps()`:

```vue
<script setup>
const props = defineProps(['foo'])

console.log(props.foo)
</script>
```

En los componentes que no son `<script setup>`, los props se declaran mediante la opción [`props`](/api/options-state.html#props):

```js
export default {
  props: ['foo'],
  setup(props) {
    // setup() recibe los props como primer argumento.
    console.log(props.foo)
  }
}
```

Observar que el argumento pasado a `defineProps()` es el mismo que el valor proporcionado a las opciones de `props`: la misma API de opciones de props se comparte entre los dos tipos de declaración.

</div>

<div class="options-api">

Los props se declaran con la opción [`props`](/api/options-state.html#props):

```js
export default {
  props: ['foo'],
  created() {
    // las props se añaden tras el this
    console.log(this.foo)
  }
}
```

</div>

Además de declarar props usando una matriz, también podemos usar la sintaxis de objeto:

<div class="options-api">

```js
export default {
  props: {
    title: String,
    likes: Number
  }
}
```

</div>
<div class="composition-api">

```js
// in <script setup>
defineProps({
  title: String,
  likes: Number
})
```

```js
// in non-<script setup>
export default {
  props: {
    title: String,
    likes: Number
  }
}
```

</div>
  
En la sintaxis de declaración de objetos, para cada propiedad, la clave es el nombre de la propiedad, mientras que el valor debe ser la función constructora del tipo esperado.

Esto no solo documenta el componente, sino que también advertirá a otros desarrolladores que usan este componente en la consola del navegador, si pasan el tipo incorrecto. Discutiremos más detalles sobre [validación de prop](#prop-validation) más adelante en esta página.

<div class="options-api">

Ver también: [Typing Component Props](/guide/typescript/options-api.html#typing-component-props) <sup class="vt-badge ts" />

</div>

<div class="composition-api">

Si se usa TypeScript con `<configuración de script>`, también es posible declarar props usando anotaciones de tipo puro:

```vue
<script setup lang="ts">
defineProps<{
  title?: string
  likes?: number
}>()
</script>
```

Más detalles: [Typing Component Props](/guide/typescript/composition-api.html#typing-component-props) <sup class="vt-badge ts" />

</div>

## Detalles en la declaración de props

### Distinción entre mayúsculas y minúsculas

Declaramos los nombres de props largos usando camelCase, porque esto evita tener que usar comillas cuando los usamos como claves de propiedad y nos permite hacer referencia a ellos directamente en expresiones de plantilla, ya que son identificadores de JavaScript válidos:

<div class="composition-api">

```js
defineProps({
  mensajeSaludo: String
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    mensajeSaludo: String
  }
}
```

</div>

```vue-html
<span>{{ mensajeSaludo }}</span>
```

Técnicamente, también se puede usar camelCase al pasar props a un componente secundario (excepto en [plantillas DOM](/guide/essentials/component-basics.html#dom-template-parsing-caveats)). Sin embargo, la convención usa kebab-case en todos los casos para alinearse con los atributos HTML:

```vue-html
<MyComponent mensaje-saludo="hola" />
```

Usamos [PascalCase para etiquetas de componentes](/guide/components/registration.html#component-name-casing) cuando sea posible, porque mejora la legibilidad de la plantilla al diferenciar los componentes de Vue de los elementos nativos. Sin embargo, no hay tantos beneficios prácticos al usar camelCase al pasar props, por lo que elegimos seguir las convenciones de cada lenguaje.

### Props Estáticos vs. Dinámicos

Hasta ahora, hemos visto props pasados como valores estáticos, como en:

```vue-html
<BlogPost title="Mi día a día con Vue" />
```

También hemos visto props asignados dinámicamente con `v-bind` o su atajo `:`, como en:

```vue-html
<!-- Asignación dinámica del valor de una variable -->
<BlogPost :title="post.title" />

<!-- Asignación dinámica del valor de una expresión compleja -->
<BlogPost :title="post.title + ' by ' + post.author.name" />
```

### Pasar diferentes tipos de valores

En los dos ejemplos anteriores, pasamos valores de cadena, pero _cualquier_ tipo de valor se puede pasar a un prop.

#### Números

```vue-html
<!-- Aunque `42` es estático, necesitamos v-bind para decirle a Vue que -->
<!-- esta es una expresión de JavaScript en lugar de una cadena.       -->
<BlogPost :likes="42" />

<!-- Asignar dinámicamente el valor de una variable. -->
<BlogPost :likes="post.likes" />
```

#### Booleanos

```vue-html
<!-- Incluir la prop sin valor implica "verdadero". -->
<BlogPost is-published />

<!-- Aunque `falso` es estático, necesitamos v-bind para decirle a Vue que -->
<!-- esta es una expresión de JavaScript en lugar de una cadena.        -->
<BlogPost :is-published="false" />

<!-- Asignar dinámicamente el valor de una variable. -->
<BlogPost :is-published="post.isPublished" />
```

#### Matrices

```vue-html
<!-- Aunque una matriz es estática, necesitamos v-bind para decirle a Vue que -->
<!-- es una expresión de JavaScript en lugar de una cadena.           -->
<BlogPost :comment-ids="[234, 266, 273]" />

<!-- Asignar dinámicamente el valor de una variable. -->
<BlogPost :comment-ids="post.commentIds" />
```

#### Objectos

```vue-html
<!-- Aunque un objeto es estático, necesitamos v-bind para decirle a Vue que -->
<!-- es una expresión de JavaScript en lugar de una cadena.           -->
<BlogPost
  :author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
 />

<!-- Asignar dinámicamente el valor de una variable. -->
<BlogPost :author="post.author" />
```

### Vinculación de varias propiedades mediante un objeto

Si queremos pasar todas las propiedadess de un objeto como props, se puede usar [`v-bind` sin argumento](/guide/essentials/template-syntax.html#dynamically-binding-multiple-attributes) (`v -bind` en lugar de `:prop-name`). Por ejemplo, dado un objeto `post`:

<div class="options-api">

```js
export default {
  data() {
    return {
      post: {
        id: 1,
        title: 'My Journey with Vue'
      }
    }
  }
}
```

</div>
<div class="composition-api">

```js
const post = {
  id: 1,
  title: 'My Journey with Vue'
}
```

</div>

La siguiente plantilla:

```vue-html
<BlogPost v-bind="post" />
```

Es equivalente a:

```vue-html
<BlogPost :id="post.id" :title="post.title" />
```

## Flujo de datos unidireccional

Todos los props forman un **enlace unidireccional** entre la propiedad secundaria y la principal: cuando la propiedad principal se actualiza, fluirá hacia la secundaria, pero no al revés. Esto evita que los componentes secundarios cambien accidentalmente el estado de los principales, lo que puede hacer que el flujo de datos de su aplicación sea más difícil de entender.

Además, cada vez que se actualice el componente principal, todas las propiedades del componente secundario se actualizarán con el valor más reciente. Esto significa que **no** debe intentar mutar un accesorio dentro de un componente secundario. Si lo hace, Vue le avisará en la consola:

<div class="composition-api">

```js
const props = defineProps(['foo'])

// ❌ warning, props are readonly!
props.foo = 'bar'
```

</div>
<div class="options-api">

```js
export default {
  props: ['foo'],
  created() {
    // ❌ warning, props are readonly!
    this.foo = 'bar'
  }
}
```

</div>

Por lo general, hay dos casos en los que es tentador mutar un prop:

1. **El prop se usa para pasar un valor inicial; el componente secundario lo usará posteriormente como una propiedad local. ** En este caso, es mejor definir una propiedad local que use el prop como su valor inicial:

   <div class="composition-api">

   ```js
   const props = defineProps(['contadorInicio'])

   // contador solo utiliza props.contadorInicio como valor inicial;
   // está desconectado de futuras actualizaciones de props.
   const counter = ref(props.contadorInicio)
   ```

   </div>
   <div class="options-api">

   ```js
   export default {
     props: ['contadorInicio'],
     data() {
       return {
         // contador solo utiliza props.contadorInicio como valor inicial;
         // está desconectado de futuras actualizaciones de props.
         counter: this.contadorInicio
       }
     }
   }
   ```

   </div>

2. **The prop is passed in as a raw value that needs to be transformed.** In this case, it's best to define a computed property using the prop's value:
2. **La propiedad se pasa como un valor sin formato que debe transformarse.** En este caso, es mejor definir una propiedad computada usando el valor del prop:

   <div class="composition-api">

   ```js
   const props = defineProps(['size'])

   // propiedad computada que se actualiza automáticamente cuando cambia la prop
   const normalizedSize = computed(() => props.size.trim().toLowerCase())
   ```

   </div>
   <div class="options-api">

   ```js
   export default {
     props: ['size'],
     computed: {
       // propiedad computada que se actualiza automáticamente cuando cambia la prop
       normalizedSize() {
         return this.size.trim().toLowerCase()
       }
     }
   }
   ```

   </div>

### Mutar objetos / Matrices Props
  
Cuando los objetos y las matrices se pasan como props, aunque que el componente secundario no puede mutar el enlace de la propiedad, si **podrá** mutar las propiedades anidadas del objeto o la matriz. Esto se debe a que en JavaScript los objetos y las matrices se pasan por referencia, y es excesivamente costoso para Vue evitar tales mutaciones.

El principal inconveniente de tales mutaciones es que permite que el componente secundario afecte el estado principal de una manera que no es obvia para este, lo que podría dificultar el razonamiento sobre el flujo de datos en el futuro. Como práctica recomendada, debe evitar este tipo de mutaciones a menos que el padre y el hijo estén estrechamente acoplados por diseño. En la mayoría de los casos, el hijo debería [emitir un evento](/guide/components/events.html) para permitir que el padre realice la mutación.

## Validación de props

Components can specify requirements for their props, such as the types you've already seen. If a requirement is not met, Vue will warn you in the browser's JavaScript console. This is especially useful when developing a component that is intended to be used by others.

To specify prop validations, you can provide an object with validation requirements to the <span class="composition-api">`defineProps()` macro</span><span class="options-api">`props` option</span>, instead of an array of strings. For example:
  
Los componentes pueden especificar requisitos para sus props, como los tipos, que ya hemos visto. Si no se cumple un requisito, Vue le avisará por la consola JavaScript del navegador. Esto es especialmente útil cuando se desarrolla un componente destinado a ser utilizado por otros.

Para especificar validaciones de props, podemos proporcionar un objeto con requisitos de validación para la <span class="composition-api">macro`defineProps()`</span><span class="options-api">opción`props`< /span>, en lugar de una matriz. Por ejemplo:

<div class="composition-api">

```js
defineProps({
  // Comprobación de tipo básico:
  // (valores `null` y `undefined` admiten cualquier tipo)
  propA: Number,
  // Múltiples tipos posibles
  propB: [String, Number],
  // Requerido string
  propC: {
    type: String,
    required: true
  },
  // Number con valor por defecto
  propD: {
    type: Number,
    default: 100
  },
  // Object con valor por defecto
  propE: {
    type: Object,
    // Por defecto, se devuelve un objeto o matriz de
    // una función factory
    default() {
      return { message: 'hola' }
    }
  },
  // Función de validación personalizada
  propF: {
    validator(value) {
      // El valor debe coincidir con una de estas cadenas
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  // Función con un valor por defecto
  propG: {
    type: Function,
    // A diferencia de los valores predeterminados de objeto o matriz, esta no es una función factory; es una función que sirve como valor predeterminado
    default() {
      return 'Default function'
    }
  }
})
```

:::tip
El código dentro del argumento `defineProps()` **no puede acceder a otras variables declaradas en `<script setup>`**, porque la expresión completa se mueve a un ámbito de función externo cuando se compila.
:::

</div>
<div class="options-api">

```js
export default {
  props: {
    // Comprobación de tipo básico
    // (valores `null` y `undefined` admiten cualquier tipo)
    propA: Number,
    // Múltiples tipos posibles
    propB: [String, Number],
    // string requerida
    propC: {
      type: String,
      required: true
    },
    // Número con valor por defecto
    propD: {
      type: Number,
      default: 100
    },
    // Objecto con valor por defecto
    propE: {
      type: Object,
      // Por defecto, se devuelve un objeto o matriz de
      // una función factory. La función recibe las props 
      // crudas que ha recibido el componente como argumento
      default(rawProps) {
        // default function receives the raw props object as argument
        return { message: 'hello' }
      }
    },
    // Custom validator function
    propF: {
      validator(value) {
        // The value must match one of these strings
        return ['success', 'warning', 'danger'].includes(value)
      }
    },
    // Function with a default value
    propG: {
      type: Function,
      // Unlike object or array default, this is not a factory function - this is a function to serve as a default value
      default() {
        return 'Default function'
      }
    }
  }
}
```

</div>

Additional details:

- All props are optional by default, unless `required: true` is specified.

- An absent optional prop will have `undefined` value.

- If a `default` value is specified, it will be used if the resolved prop value is `undefined` - this includes both when the prop is absent, or an explicit `undefined` value is passed.

When prop validation fails, Vue will produce a console warning (if using the development build).

<div class="composition-api">

If using [Type-based props declarations](/api/sfc-script-setup.html#typescript-only-features), Vue will try its best to compile the type annotations into equivalent runtime prop declarations. For example, `defineProps<{ msg: string }>` will be compiled into `{ msg: { type: String, required: true }}`.

</div>
<div class="options-api">

::: tip Note
Note that props are validated **before** a component instance is created, so instance properties (e.g. `data`, `computed`, etc.) will not be available inside `default` or `validator` functions.
:::

</div>

### Runtime Type Checks

The `type` can be one of the following native constructors:

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Date`
- `Function`
- `Symbol`

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
}
```

</div>

The component can be used like this:

```vue-html
<!-- equivalent of passing :disabled="true" -->
<MyComponent disabled />

<!-- equivalent of passing :disabled="false" -->
<MyComponent />
```

When a prop is declared to allow multiple types, e.g.

<div class="composition-api">

```js
defineProps({
  disabled: [Boolean, Number]
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    disabled: [Boolean, Number]
  }
}
```

</div>

The casting rules for `Boolean` will apply regardless of type appearance order.
