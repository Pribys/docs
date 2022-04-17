# Eventos de componentes

> Se supone que ya conoces lo tratado en [Lo básico sobre componentes](/guide/essentials/component-basics). Si eres novato, deberías leer primero esa sección.

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/defining-custom-events-emits" title="Free Vue.js Lesson on Defining Custom Events"/>
</div>

## Emitir y escuchar eventos

Un componente puede emitir eventos propios directamente en expresiones de plantilla (por ejemplo, en un controlador `v-on`) usando la función `$emit`:

```vue-html
<!-- MiComponente -->
<button @click="$emit('algunEvento')">Pulsar</button>
```

<div class="options-api">

La función `$emit()` también está disponible en la instancia del componente como `this.$emit()`.

</div>

El componente padre puede escucharlo usando `v-on`:

```vue-html
<MiComponente @some-event="callback" />
```

The `.once` modifier is also supported on component event listeners:

```vue-html
<MyComponent @algun-evento.once="callback" />
```

Al igual que los componentes y props, los nombres de eventos proporcionan una transformación automática de tipografía. Debemos saber que si emitimos un evento camelCase, podemos escucharlo usando un oyente kebab-cased en el padre. Al igual que con [Distinción entre mayúsculas y minúsculas](/guide/components/props.html#detalles-en-la-declaracion-de-props),  en las plantillas recomendamos usar oyentes de eventos en formato kebab-case.

:::tip
A diferencia de los eventos DOM nativos, los eventos emitidos por componentes **no** se propagan. Solo se puede escuchar los eventos emitidos por un componente secundario directo.
:::

## Argumentos de un evento

A veces, es útil emitir un valor específico con un evento. Por ejemplo, podemos querer que el componente `<BlogPost>` se encargue de cuánto agrandar el texto. En esos casos, podemos pasar argumentos adicionales a `$emit` para proporcionar este valor:

```vue-html
<button @click="$emit('incrementarEn', 1)">
  Aumenta 1
</button>
```

Luego, cuando escuchamos el evento en el componente padre, podemos usar una función de flecha como oyente, lo que nos permite acceder al argumento del evento:

```vue-html
<MiBoton @incrementar-en="(n) => contador += n" />
```

O, si el controlador del evento es un método:

```vue-html
<MiBoton @incrementar-en="aumentarContador" />
```

Así, el valor se pasará como el primer parámetro de ese método:

<div class="options-api">

```js
methods: {
  aumentarContador(n) {
    this.contador += n
  }
}
```

</div>
<div class="composition-api">

```js
function aumentarContador(n) {
  contador.value += n
}
```

</div>

:::tip
Todos los argumentos adicionales pasados a `$emit()` después del nombre del evento se reenviarán al oyente. Por ejemplo, con `$emit('foo', 1, 2, 3)` la función oyente recibirá tres argumentos.
:::

## Declarar los eventos emitidos

Los eventos emitidos se pueden declarar explícitamente en el componente mediante la opción <span class="options-api">[`emits`](/api/options-state.html#emits) </span>.

<div class="composition-api">

```vue
<script setup>
const emit = defineEmits(['inFocus', 'submit'])
</script>
```

La función `emit` devuelta se puede usar para emitir eventos en JavaScript.

Si no se usa `<script setup>`, los eventos deben declararse usando la opción [`emits`](/api/options-state.html#emits), y la función `emit` se expone en el contexto `setup()`:

```js
export default {
  emits: ['inFocus', 'submit'],
  setup(props, ctx) {
    ctx.emit('submit')
  }
}
```

</div>
<div class="options-api">

```js
export default {
  emits: ['inFocus', 'submit']
}
```

</div>

La opción `emits` también admite una sintaxis de objeto, lo que nos permite realizar una validación en tiempo de ejecución del contenido de los eventos emitidos:

<div class="composition-api">

```vue
<script setup>
const emit = defineEmits({
  submit(payload) {
    // devuelve `true` o `false` para indicar
    // si la validación pasa / falla
  }
})
</script>
```

Si se usa TypeScript con `<script setup>`, también es posible declarar eventos emitidos usando anotaciones de tipo puro:

```vue
<script setup lang="ts">
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()
</script>
```

Más detalles: [Typing Component Emits](/guide/typescript/composition-api.html#typing-component-emits) <sup class="vt-badge ts" />

</div>
<div class="options-api">

```js
export default {
  emits: {
    submit(payload) {
      // devuelve `true` o `false` para indicar
      // si la validación pasa / falla
    }
  }
}
```

Ver también: [Typing Component Emits](/guide/typescript/options-api.html#typing-component-emits) <sup class="vt-badge ts" />

</div>

Aunque es opcional, se recomienda definir todos los eventos emitidos para documentar mejor cómo debería funcionar un componente. También permite que Vue excluya oyentes conocidos de [atributos fallidos](/guide/components/attrs.html#v-on-listener-inheritance).

:::tip
Si se define un evento nativo (por ejemplo, `click`) en la opción `emits`, el oyente ahora solo escuchará los eventos `click` emitidos por el componente y ya no responderá a los eventos `click` nativos.
:::

## Validación de eventos

De manera similar a la validación de tipo de un prop, un evento emitido se puede validar si se define con la sintaxis de objeto en lugar de la sintaxis de matriz.

Para agregar validación, al evento se le asigna una función que recibe los argumentos pasados a la <span class="options-api">`this.$emit`</span><span class="composition-api">`emit`</span> llamada y devuelve un valor booleano para indicar si el evento es válido o no.

<div class="composition-api">

```vue
<script setup>
const emit = defineEmits({
  // Sin validación
  click: null,

  // Validación de evento emitido
  submit: ({ email, password }) => {
    if (email && password) {
      return true
    } else {
      console.warn('Invalid submit event payload!')
      return false
    }
  }
})

function submitForm(email, password) {
  emit('submit', { email, password })
}
</script>
```

</div>
<div class="options-api">

```js
export default {
  emits: {
    // Sin validación
    click: null,

    // Validación de evento emitido
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false
      }
    }
  },
  methods: {
    submitForm(email, password) {
      this.$emit('submit', { email, password })
    }
  }
}
```

</div>

## Uso con `v-model`

Los eventos también se pueden usar para crear entradas de texto que funcionen con `v-model`. Recuerda:

```vue-html
<input v-model="searchText" />
```

hace lo mismo que:

```vue-html
<input
  :value="searchText"
  @input="searchText = $event.target.value"
/>
```

Cuando se usa en un componente, `v-model` funciona así:

```vue-html
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
```
  
Sin embargo, para que esto realmente funcione, el `<input>` dentro del componente debe:

- Vincular el atributo `value` al prop `modelValue`
- Desde `input`, emitir un evento `update:modelValue` con el nuevo valor

Lo vemos en acción:

<div class="options-api">

```vue
<!-- CustomInput.vue -->
<script>
export default {
  props: ['modelValue'],
  emits: ['update:modelValue']
}
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>
<div class="composition-api">

```vue
<!-- CustomInput.vue -->
<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>

Ahora `v-model` debería funcionar perfectamente con este componente:

```vue-html
<CustomInput v-model="searchText" />
```

<div class="options-api">

[Try it in the Playground](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBDdXN0b21JbnB1dCBmcm9tICcuL0N1c3RvbUlucHV0LnZ1ZSdcblxuZXhwb3J0IGRlZmF1bHQge1xuICBjb21wb25lbnRzOiB7IEN1c3RvbUlucHV0IH0sXG4gIGRhdGEoKSB7XG4gICAgcmV0dXJuIHtcbiAgICAgIG1lc3NhZ2U6ICdoZWxsbydcbiAgICB9XG4gIH1cbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxDdXN0b21JbnB1dCB2LW1vZGVsPVwibWVzc2FnZVwiIC8+IHt7IG1lc3NhZ2UgfX1cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIkN1c3RvbUlucHV0LnZ1ZSI6IjxzY3JpcHQ+XG5leHBvcnQgZGVmYXVsdCB7XG4gIHByb3BzOiBbJ21vZGVsVmFsdWUnXSxcbiAgZW1pdHM6IFsndXBkYXRlOm1vZGVsVmFsdWUnXVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGlucHV0XG4gICAgOnZhbHVlPVwibW9kZWxWYWx1ZVwiXG4gICAgQGlucHV0PVwiJGVtaXQoJ3VwZGF0ZTptb2RlbFZhbHVlJywgJGV2ZW50LnRhcmdldC52YWx1ZSlcIlxuICAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>
<div class="composition-api">

[Try it in the Playground](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBDdXN0b21JbnB1dCBmcm9tICcuL0N1c3RvbUlucHV0LnZ1ZSdcbiAgXG5jb25zdCBtZXNzYWdlID0gcmVmKCdoZWxsbycpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8Q3VzdG9tSW5wdXQgdi1tb2RlbD1cIm1lc3NhZ2VcIiAvPiB7eyBtZXNzYWdlIH19XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJDdXN0b21JbnB1dC52dWUiOiI8c2NyaXB0IHNldHVwPlxuZGVmaW5lUHJvcHMoWydtb2RlbFZhbHVlJ10pXG5kZWZpbmVFbWl0cyhbJ3VwZGF0ZTptb2RlbFZhbHVlJ10pXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXRcbiAgICA6dmFsdWU9XCJtb2RlbFZhbHVlXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOm1vZGVsVmFsdWUnLCAkZXZlbnQudGFyZ2V0LnZhbHVlKVwiXG4gIC8+XG48L3RlbXBsYXRlPiJ9)

</div>

Otra forma de implementar `v-model` dentro de este componente es usar una propiedad `computed` escribible con un getter y un setter. El método `get` debería devolver la propiedad `modelValue` y el método `set` debería emitir el evento correspondiente:

<div class="options-api">

```vue
<!-- CustomInput.vue -->
<script>
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  computed: {
    value: {
      get() {
        return this.modelValue
      },
      set(value) {
        this.$emit('update:modelValue', value)
      }
    }
  }
}
</script>

<template>
  <input v-model="value" />
</template>
```

</div>
<div class="composition-api">

```vue
<!-- CustomInput.vue -->
<script setup>
import { computed } from 'vue'

const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])

const value = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emit('update:modelValue', value)
  }
})
</script>

<template>
  <input v-model="value" />
</template>
```

</div>

### Argumentos en `v-model`

Por defecto, `v-model` en un componente usa `modelValue` como prop y `update:modelValue` como evento. Podemos modificar estos nombres pasando un argumento a `v-model`:

```vue-html
<MiComponente v-model:title="Título" />
```

En este caso, el componente secundario debe esperar un prop `title` y emitir un evento `update:title` para actualizar el valor del componente principal:

<div class="composition-api">

```vue
<!-- MiComponente.vue -->
<script setup>
defineProps(['title'])
defineEmits(['update:title'])
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
```

[Prueba en la zona de práctica](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcbiAgXG5jb25zdCB0aXRsZSA9IHJlZigndi1tb2RlbCBhcmd1bWVudCBleGFtcGxlJylcbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxoMT57eyB0aXRsZSB9fTwvaDE+XG4gIDxNeUNvbXBvbmVudCB2LW1vZGVsOnRpdGxlPVwidGl0bGVcIiAvPlxuPC90ZW1wbGF0ZT4iLCJpbXBvcnQtbWFwLmpzb24iOiJ7XG4gIFwiaW1wb3J0c1wiOiB7XG4gICAgXCJ2dWVcIjogXCJodHRwczovL3NmYy52dWVqcy5vcmcvdnVlLnJ1bnRpbWUuZXNtLWJyb3dzZXIuanNcIlxuICB9XG59IiwiTXlDb21wb25lbnQudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmRlZmluZVByb3BzKFsndGl0bGUnXSlcbmRlZmluZUVtaXRzKFsndXBkYXRlOnRpdGxlJ10pXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXRcbiAgICB0eXBlPVwidGV4dFwiXG4gICAgOnZhbHVlPVwidGl0bGVcIlxuICAgIEBpbnB1dD1cIiRlbWl0KCd1cGRhdGU6dGl0bGUnLCAkZXZlbnQudGFyZ2V0LnZhbHVlKVwiXG4gIC8+XG48L3RlbXBsYXRlPiJ9)

</div>
<div class="options-api">

```vue
<!-- MyComponent.vue -->
<script>
export default {
  props: ['title'],
  emits: ['update:title']
}
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
```

[Prueba en la zona de práctica](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcblxuZXhwb3J0IGRlZmF1bHQge1xuICBjb21wb25lbnRzOiB7IE15Q29tcG9uZW50IH0sXG4gIGRhdGEoKSB7XG4gICAgcmV0dXJuIHtcbiAgICAgIHRpdGxlOiAndi1tb2RlbCBhcmd1bWVudCBleGFtcGxlJ1xuICAgIH1cbiAgfVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGgxPnt7IHRpdGxlIH19PC9oMT5cbiAgPE15Q29tcG9uZW50IHYtbW9kZWw6dGl0bGU9XCJ0aXRsZVwiIC8+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJNeUNvbXBvbmVudC52dWUiOiI8c2NyaXB0PlxuZXhwb3J0IGRlZmF1bHQge1xuICBwcm9wczogWyd0aXRsZSddLFxuICBlbWl0czogWyd1cGRhdGU6dGl0bGUnXVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgPGlucHV0XG4gICAgdHlwZT1cInRleHRcIlxuICAgIDp2YWx1ZT1cInRpdGxlXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOnRpdGxlJywgJGV2ZW50LnRhcmdldC52YWx1ZSlcIlxuICAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>

### Multiple enlaces `v-model` 

Para aprovechar la capacidad de apuntar a un prop y evento en particular, como hemos aprendido en [argumentos en `v-model`] (#v-model-arguments), ahora podemos crear múltiples enlaces de v-model en una sola instancia de componente.

Cada v-model se sincronizará con un prop diferente, sin necesidad de opciones adicionales en el componente:

```vue-html
<UserName
  v-model:nombre="nombre"
  v-model:apellido="apellido"
/>
```

<div class="composition-api">

```vue
<script setup>
defineProps({
  nombre: String,
  apellido: String
})

defineEmits(['update:nombre', 'update:apellido'])
</script>

<template>
  <input
    type="text"
    :value="nombre"
    @input="$emit('update:nombre', $event.target.value)"
  />
  <input
    type="text"
    :value="apellido"
    @input="$emit('update:apellido', $event.target.value)"
  />
</template>
```

[Prueba en la zona de práctica](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBVc2VyTmFtZSBmcm9tICcuL1VzZXJOYW1lLnZ1ZSdcblxuY29uc3QgZmlyc3ROYW1lID0gcmVmKCdKb2huJylcbmNvbnN0IGxhc3ROYW1lID1yZWYoJ0RvZScpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aDE+e3sgZmlyc3ROYW1lIH19IHt7IGxhc3ROYW1lIH19PC9oMT5cbiAgPFVzZXJOYW1lXG4gICAgdi1tb2RlbDpmaXJzdC1uYW1lPVwiZmlyc3ROYW1lXCJcbiAgICB2LW1vZGVsOmxhc3QtbmFtZT1cImxhc3ROYW1lXCJcbiAgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIlVzZXJOYW1lLnZ1ZSI6IjxzY3JpcHQgc2V0dXA+XG5kZWZpbmVQcm9wcyh7XG4gIGZpcnN0TmFtZTogU3RyaW5nLFxuICBsYXN0TmFtZTogU3RyaW5nXG59KVxuXG5kZWZpbmVFbWl0cyhbJ3VwZGF0ZTpmaXJzdE5hbWUnLCAndXBkYXRlOmxhc3ROYW1lJ10pXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXRcbiAgICB0eXBlPVwidGV4dFwiXG4gICAgOnZhbHVlPVwiZmlyc3ROYW1lXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOmZpcnN0TmFtZScsICRldmVudC50YXJnZXQudmFsdWUpXCJcbiAgLz5cbiAgPGlucHV0XG4gICAgdHlwZT1cInRleHRcIlxuICAgIDp2YWx1ZT1cImxhc3ROYW1lXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOmxhc3ROYW1lJywgJGV2ZW50LnRhcmdldC52YWx1ZSlcIlxuICAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>
<div class="options-api">

```vue
<script>
export default {
  props: {
    nombre: String,
    apellido: String
  },
  emits: ['update:nombre', 'update:apellido']
}
</script>

<template>
  <input
    type="text"
    :value="nombre"
    @input="$emit('update:nombre', $event.target.value)"
  />
  <input
    type="text"
    :value="apellido"
    @input="$emit('update:apellido', $event.target.value)"
  />
</template>
```

[Prueba en la zona de práctica](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBVc2VyTmFtZSBmcm9tICcuL1VzZXJOYW1lLnZ1ZSdcblxuZXhwb3J0IGRlZmF1bHQge1xuICBjb21wb25lbnRzOiB7IFVzZXJOYW1lIH0sXG4gIGRhdGEoKSB7XG4gICAgcmV0dXJuIHtcbiAgICAgIGZpcnN0TmFtZTogJ0pvaG4nLFxuICAgICAgbGFzdE5hbWU6ICdEb2UnXG4gICAgfVxuICB9XG59XG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aDE+e3sgZmlyc3ROYW1lIH19IHt7IGxhc3ROYW1lIH19PC9oMT5cbiAgPFVzZXJOYW1lXG4gICAgdi1tb2RlbDpmaXJzdC1uYW1lPVwiZmlyc3ROYW1lXCJcbiAgICB2LW1vZGVsOmxhc3QtbmFtZT1cImxhc3ROYW1lXCJcbiAgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIlVzZXJOYW1lLnZ1ZSI6IjxzY3JpcHQ+XG5leHBvcnQgZGVmYXVsdCB7XG4gIHByb3BzOiB7XG5cdCAgZmlyc3ROYW1lOiBTdHJpbmcsXG4gIFx0bGFzdE5hbWU6IFN0cmluZ1xuXHR9LFxuICBlbWl0czogWyd1cGRhdGU6Zmlyc3ROYW1lJywgJ3VwZGF0ZTpsYXN0TmFtZSddXG59XG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXRcbiAgICB0eXBlPVwidGV4dFwiXG4gICAgOnZhbHVlPVwiZmlyc3ROYW1lXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOmZpcnN0TmFtZScsICRldmVudC50YXJnZXQudmFsdWUpXCJcbiAgLz5cbiAgPGlucHV0XG4gICAgdHlwZT1cInRleHRcIlxuICAgIDp2YWx1ZT1cImxhc3ROYW1lXCJcbiAgICBAaW5wdXQ9XCIkZW1pdCgndXBkYXRlOmxhc3ROYW1lJywgJGV2ZW50LnRhcmdldC52YWx1ZSlcIlxuICAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>

### Manejo de modificadores `v-model`

Cuando estábamos aprendiendo sobre los campos de entrada de formulario, vimos que `v-model` tiene [modificadores incorporados] (/guide/essentials/forms.html#modifiers) - `.trim`, `.number` y `.lazy `. Sin embargo, en algunos casos, es posible que prefieras utilizar tus propios modificadores personalizados.

Vamos a crear el modificador personalizado de ejemplo, `mayúsculas`, que pone en mayúscula la primera letra de la cadena proporcionada por el enlace `v-model`:

```vue-html
<MiComponente v-model.mayuscula="miTexto" />
```

Los modificadores agregados a un componente `v-model` se proporcionarán al componente a través de la propiedad `modelModifiers`. En el siguiente ejemplo, hemos creado un componente que contiene una prop `modelModifiers` que, por defecto, es un objeto vacío:

<div class="composition-api">

```vue{4,9}
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) }
})

defineEmits(['update:modelValue'])

console.log(props.modelModifiers) // { mayuscula: true }
</script>

<template>
  <input
    type="text"
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>
<div class="options-api">

```vue{11}
<script>
export default {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  created() {
    console.log(this.modelModifiers) // { mayuscula: true }
  }
}
</script>

<template>
  <input
    type="text"
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

</div>

Observar que la propiedad `modelModifiers` del componente contiene `mayúscula` y su valor es `true`, debido a que se configuró en el enlace `v-model` `v-model.mayuscula="miTexto"`.

Ahora que tenemos configurado nuestro prop, podemos verificar las claves del objeto `modelModifiers` y escribir un controlador para cambiar el valor emitido. En el siguiente código, pondremos en mayúsculas la cadena cada vez que el elemento `<input />` active un evento `input`.

<div class="composition-api">

```vue{11-13}
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) }
})

const emit = defineEmits(['update:modelValue'])

function emitValue(e) {
  let value = e.target.value
  if (props.modelModifiers.mayuscula) {
    value = value.charAt(0).toUpperCase() + value.slice(1)
  }
  emit('update:modelValue', value)
}
</script>

<template>
  <input type="text" :value="modelValue" @input="emitValue" />
</template>
```

[Prueba en la zona de práctica](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdCBzZXR1cD5cbmltcG9ydCB7IHJlZiB9IGZyb20gJ3Z1ZSdcbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcbiAgXG5jb25zdCBteVRleHQgPSByZWYoJycpXG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICBUaGlzIGlucHV0IGNhcGl0YWxpemVzIGV2ZXJ5dGhpbmcgeW91IGVudGVyOlxuICA8TXlDb21wb25lbnQgdi1tb2RlbC5jYXBpdGFsaXplPVwibXlUZXh0XCIgLz5cbjwvdGVtcGxhdGU+IiwiaW1wb3J0LW1hcC5qc29uIjoie1xuICBcImltcG9ydHNcIjoge1xuICAgIFwidnVlXCI6IFwiaHR0cHM6Ly9zZmMudnVlanMub3JnL3Z1ZS5ydW50aW1lLmVzbS1icm93c2VyLmpzXCJcbiAgfVxufSIsIk15Q29tcG9uZW50LnZ1ZSI6IjxzY3JpcHQgc2V0dXA+XG5jb25zdCBwcm9wcyA9IGRlZmluZVByb3BzKHtcbiAgbW9kZWxWYWx1ZTogU3RyaW5nLFxuICBtb2RlbE1vZGlmaWVyczogeyBkZWZhdWx0OiAoKSA9PiAoe30pIH1cbn0pXG5cbmNvbnN0IGVtaXQgPSBkZWZpbmVFbWl0cyhbJ3VwZGF0ZTptb2RlbFZhbHVlJ10pXG5cbmZ1bmN0aW9uIGVtaXRWYWx1ZShlKSB7XG4gIGxldCB2YWx1ZSA9IGUudGFyZ2V0LnZhbHVlXG4gIGlmIChwcm9wcy5tb2RlbE1vZGlmaWVycy5jYXBpdGFsaXplKSB7XG4gICAgdmFsdWUgPSB2YWx1ZS5jaGFyQXQoMCkudG9VcHBlckNhc2UoKSArIHZhbHVlLnNsaWNlKDEpXG4gIH1cbiAgZW1pdCgndXBkYXRlOm1vZGVsVmFsdWUnLCB2YWx1ZSlcbn1cbjwvc2NyaXB0PlxuXG48dGVtcGxhdGU+XG4gIDxpbnB1dCB0eXBlPVwidGV4dFwiIDp2YWx1ZT1cIm1vZGVsVmFsdWVcIiBAaW5wdXQ9XCJlbWl0VmFsdWVcIiAvPlxuPC90ZW1wbGF0ZT4ifQ==)

</div>
<div class="options-api">

```vue{13-15}
<script>
export default {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.mayuscula) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  }
}
</script>

<template>
  <input type="text" :value="modelValue" @input="emitValue" />
</template>
```

[Prueba en la zona de práctica](https://sfc.vuejs.org/#eyJBcHAudnVlIjoiPHNjcmlwdD5cbmltcG9ydCBNeUNvbXBvbmVudCBmcm9tICcuL015Q29tcG9uZW50LnZ1ZSdcbiAgXG5leHBvcnQgZGVmYXVsdCB7XG4gIGNvbXBvbmVudHM6IHsgTXlDb21wb25lbnQgfSxcbiAgZGF0YSgpIHtcbiAgICByZXR1cm4ge1xuICAgICAgbXlUZXh0OiAnJ1xuICAgIH1cbiAgfVxufVxuPC9zY3JpcHQ+XG5cbjx0ZW1wbGF0ZT5cbiAgVGhpcyBpbnB1dCBjYXBpdGFsaXplcyBldmVyeXRoaW5nIHlvdSBlbnRlcjpcbiAgPE15Q29tcG9uZW50IHYtbW9kZWwuY2FwaXRhbGl6ZT1cIm15VGV4dFwiIC8+XG48L3RlbXBsYXRlPiIsImltcG9ydC1tYXAuanNvbiI6IntcbiAgXCJpbXBvcnRzXCI6IHtcbiAgICBcInZ1ZVwiOiBcImh0dHBzOi8vc2ZjLnZ1ZWpzLm9yZy92dWUucnVudGltZS5lc20tYnJvd3Nlci5qc1wiXG4gIH1cbn0iLCJNeUNvbXBvbmVudC52dWUiOiI8c2NyaXB0PlxuZXhwb3J0IGRlZmF1bHQge1xuICBwcm9wczoge1xuICAgIG1vZGVsVmFsdWU6IFN0cmluZyxcbiAgICBtb2RlbE1vZGlmaWVyczoge1xuICAgICAgZGVmYXVsdDogKCkgPT4gKHt9KVxuICAgIH1cbiAgfSxcbiAgZW1pdHM6IFsndXBkYXRlOm1vZGVsVmFsdWUnXSxcbiAgbWV0aG9kczoge1xuICAgIGVtaXRWYWx1ZShlKSB7XG4gICAgICBsZXQgdmFsdWUgPSBlLnRhcmdldC52YWx1ZVxuICAgICAgaWYgKHRoaXMubW9kZWxNb2RpZmllcnMuY2FwaXRhbGl6ZSkge1xuICAgICAgICB2YWx1ZSA9IHZhbHVlLmNoYXJBdCgwKS50b1VwcGVyQ2FzZSgpICsgdmFsdWUuc2xpY2UoMSlcbiAgICAgIH1cbiAgICAgIHRoaXMuJGVtaXQoJ3VwZGF0ZTptb2RlbFZhbHVlJywgdmFsdWUpXG4gICAgfVxuICB9XG59XG48L3NjcmlwdD5cblxuPHRlbXBsYXRlPlxuICA8aW5wdXQgdHlwZT1cInRleHRcIiA6dmFsdWU9XCJtb2RlbFZhbHVlXCIgQGlucHV0PVwiZW1pdFZhbHVlXCIgLz5cbjwvdGVtcGxhdGU+In0=)

</div>

Para enlaces `v-model` con argumentos y modificadores, el nombre de la propiedad generada será `arg + "Modificadores"`. Por ejemplo:

```vue-html
<MiComponente v-model:title.mayuscula="miTexto">
```

The corresponding declarations should be:

<div class="composition-api">

```js
const props = defineProps(['title', 'titleModifiers'])
defineEmits(['update:title'])

console.log(props.titleModifiers) // { mayuscula: true }
```

</div>
<div class="options-api">

```js
export default {
  props: ['title', 'titleModifiers'],
  emits: ['update:title'],
  created() {
    console.log(this.titleModifiers) // { mayuscula: true }
  }
}
```

</div>
