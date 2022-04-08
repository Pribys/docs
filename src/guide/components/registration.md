# Registro de componentes

<VueSchoolLink href="https://vueschool.io/lessons/vue-3-global-vs-local-vue-components" title="Free Vue.js Component Registration Lesson"/>

> Esta página supone que ya has leído [Components Basics](/guide/essentials/component-basics). Empieza por allí si no sabes nada sobre componentes.

Un componente debe estar "registrado" para que Vue sepa dónde ubicarlo cuando lo encuentra en una plantilla. Hay dos formas de registrar componentes: global y local.

## Registro Global

Es posible tener los componentes disponibles de modo global en una [Vue application](/guide/essentials/application.html) utilizando el método `app.component()`:

```js
import { createApp } from 'vue'

const app = createApp({})

app.component(
  // el nombre registrado
  'MiComponente',
  // la implementación
  {
    /* ... */
  }
)
```

Si utilizamos CAUs, registraremos el archivo `.vue` que importamos:

```js
import MiComponente from './App.vue'

app.component('MiComponente', MiComponente)
```

El método `app.component()` se puede encadenar:

```js
app
  .component('ComponenteA', ComponenteA)
  .component('ComponenteB', ComponenteB)
  .component('ComponenteC', ComponenteC)
```

Los componentes registrados globalmente se pueden usar en la plantilla de cualquier componente dentro de la aplicación:.

```vue-html
<!-- esto funcionará en cualquier componente de la app -->
<ComponentA/>
<ComponentB/>
<ComponentC/>
```

Esto incluso se aplica a todos los subcomponentes, lo que significa que cualquiera de estos tres componentes también estarían disponibles _uno dentro del otro_.

## Registro Local

Si bien es cómodo, el registro global tiene algunos inconvenientes:

1. El registro global evita que los sistemas de compilación eliminen los componentes no utilizados (también conocido como "tree-shaking"). Al registrar globalmente un componente que, finalmente, no se utiliza en la aplicación, se incluirá este en el paquete final.

2. El registro global hace que las relaciones de dependencia sean menos explícitas en aplicaciones grandes. Hace que sea difícil ubicar la implementación de un componente secundario desde un componente principal que lo usa. Esto puede afectar la capacidad de mantenimiento a largo plazo de forma similar al uso de demasiadas variables globales.

El registro local limita la disponibilidad de los componentes registrados solo al componente actual. Hace que la relación de dependencia sea más explícita y facilita el *tree-shaking*.

<div class="composition-api">

Si utilizamos CAU con `<script setup>`, los componentes importados se registran localmente de modo automático:

```vue
<script setup>
import ComponenteA from './ComponenteA.vue'
</script>

<template>
  <ComponenteA />
</template>
```

Si no utilizamos CAU, es necesario utilizar la opción `components`:

```js
import ComponenteA from './ComponenteA.js'

export default {
  components: {
    ComponenteA
  },
  setup() {
    // ...
  }
}
```

</div>
<div class="options-api">

El registro local se hace utilizando la opción `components`:

```vue
<script>
import ComponenteA from './ComponenteA.vue'

export default {
  components: {
    ComponenteA
  }
}
</script>

<template>
  <ComponenteA />
</template>
```

</div>

Para cada propiedad del objeto `components`, la clave será el nombre registrado del componente, mientras que el valor contendrá la implementación del componente. El ejemplo anterior utiliza la abreviatura de propiedad de ES2015 y es equivalente a:

```js
export default {
  components: {
    ComponenteA: ComponenteA
  }
  // ...
}
```

Tenga en cuenta que **los componentes registrados localmente _no_ están también disponibles en los componentes descendientes**. En este caso, `ComponenteA` estará disponible solo para el componente actual, no para ninguno de sus componentes secundarios o descendientes.

## Uso de mayúsculas o minúsculas para nombrar los componentes
  
A lo largo de la guía, estamos usando nombres PascalCase al registrar componentes. Esto es porque:

1. Los nombres PascalCase son identificadores JavaScript válidos. Esto facilita la importación y el registro de componentes en JavaScript. También ayuda a los IDE con el autocompletado.

2. `<PascalCase />` hace más obvio que se trata de un componente Vue en lugar de un elemento HTML nativo en las plantillas. También diferencia los componentes Vue de los elementos personalizados (componentes web).

Este es el estilo recomendado cuando se trabaja con CAU o plantillas de cadena. Sin embargo, como se explica en [Advertencias de análisis de plantillas DOM](/guide/essentials/component-basics.html#dom-template-parsing-caveats), las etiquetas PascalCase no se pueden usar en las plantillas DOM.

Afortunadamente, Vue admite la resolución de etiquetas kebab-case en componentes registrados con PascalCase. Esto significa que se puede hacer referencia a un componente registrado como `MiComponente` en la plantilla a través de `<MiComponente>` y `<mi-componente>`. Esto nos permite usar el mismo código de registro del componente JavaScript independientemente de la fuente de la plantilla.
