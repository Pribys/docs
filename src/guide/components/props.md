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

