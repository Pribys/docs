---
outline: deep
---

# Atributos perdidos (Fallthrough)

> Se supone que ya conoces lo tratado en [lo básico sobre componentes](/guide/essentials/component-basics). Si eres novato, deberías leer primero esa sección.

## Herencia de atributos

Un "atributo perdido" (o "fallthrough attribute" en inglés) es un atributo u oyente `v-on` que se pasa a un componente, pero que no se declara explícitamente en los [props](./props) o [emits](./events.html#declarar-los-eventos-emitidos) del componente receptor. Los ejemplos comunes de esto incluyen los atributos `class`, `estyle` e `id`.

Cuando un componente representa un único elemento raíz, los atributos perdidos se agregarán automáticamente a los atributos del elemento raíz. Por ejemplo, dado un componente `<MiBoton>` con la siguiente plantilla:

```vue-html
<!-- template of <MiBoton> -->
<button>Pulsar</button>
```

And a parent using this component with:

```vue-html
<MiBoton class="large" />
```

The final rendered DOM would be:

```html
<button class="large">pulsar</button>
```

### Fusión de `class` y `style`

Si el elemento raíz del componente secundario ya tiene atributos `class` o `style` existentes, se fusionará con los valores `class` y `style` que se heredan del padre. Supongamos que cambiamos la plantilla de `<MiBoton>` en el ejemplo anterior a:

```vue-html
<!-- template of <MiBoton> -->
<button class="btn">pulsar</button>
```

Entonces el DOM renderizado final se convertiría en:

```html
<button class="btn large">pulsar</button>
```

### Herencia del oyente `v-on`

La misma regla se aplica a los oyentes de eventos `v-on`:

```vue-html
<MiBoton @click="onClick" />
```

El oyente `click` se agregará al elemento raíz de `<MiBoton>`, es decir, el elemento nativo `<button>`. Cuando se hace clic en el `<button>` nativo, activará el método `onClick` del componente principal. Si el `<button>` nativo ya tiene un oyente `click` vinculado con `v-on`, ambos oyentes se activarán.

### Herencia de componentes anidados

Si un componente representa a otro componente como su nodo raíz, por ejemplo, si refactorizamos `<Miboton>` para representar un `<BotonBase>` como su raíz:

```vue-html
<!-- plantilla de <MiBoton/> que simplemente representa otro componente -->
<BotonBase />
```

Entonces, los atributos perdidos recibidos por `<MiBoton>` se reenviarán automáticamente a `<BotonBase>`.

Debe tenerse en cuenta que:

1. Los atributos reenviados no incluyen ningún atributo declarado como props, u oyentes `v-on` de eventos declarados por `<MiBoton>`. En otras palabras, los props y oyentes declarados han sido "consumidos" por `<MiBoton>`.

2. Los atributos reenviados pueden ser aceptados como props por `<BotonBase>`, si así lo declara.

## Deshabilitar la herencia de atributos

Si **no** quiero que un componente herede atributos automáticamente, se puede configurar `inheritAttrs: false` en las opciones del componente.

<div class="composition-api">

Si se usa `<script setup>`, será necesario declarar esta opción usando un bloque `<script>` normal e independiente:

```vue
<script>
// uso de <script> normal para declarar opciones
export default {
  inheritAttrs: false
}
</script>

<script setup>
// ...lógica de configuración
</script>
```

</div>

El escenario común para deshabilitar la herencia de atributos es cuando los atributos deben aplicarse a otros elementos además del nodo raíz. Al establecer la opción `inheritAttrs` en `false`, podemos tomar el control total sobre dónde se deben aplicar los atributos perdidos.

Se puede acceder a estos atributos perdidos directamente en expresiones de plantilla como `$attrs`:

```vue-html
<span>Atributos fallidos: {{ $attrs }}</span>
```

El objeto `$attrs` incluye todos los atributos que no están declarados por las opciones `props` o `emits` del componente (por ejemplo, `class`, `style`, oyentes `v-on`, etc.).

Algunas notas:
  
- A diferencia de los props, los atributos perdidos conservan su escritura original en JavaScript, por lo que se debe acceder a un atributo como `foo-bar` como `$attrs['foo-bar']`.

- Un oyente de eventos `v-on` como `@click` se incluirá en el objeto como una función en `$attrs.onClick`.

Usando nuestro ejemplo de componente `<MiBoton>` de la [sección anterior](#herencia-de-atributos), a veces es posible que necesitemos envolver el elemento `<button>` real con un `<div>` adicional por motivos de estilo:

```vue-html
<div class="btn-wrapper">
  <button class="btn">pulsar</button>
</div>
```

Queremos que todos los atributos perdidos como `class` y los oyentes `v-on` se apliquen al `<button>` interior, no al `<div>` exterior. Podemos lograrlo con `inheritAttrs: false` y `v-bind="$attrs"`:

```vue-html{2}
<div class="btn-wrapper">
  <button class="btn" v-bind="$attrs">Pulsar</button>
</div>
```

Recuerde que [`v-bind` sin argumento](/guide/essentials/template-syntax.html#dynamically-binding-multiple-attributes) vincula todas las propiedades de un objeto como atributos del elemento de destino.

## Herencia de atributos en varios nodos raíz

A diferencia de los componentes con un solo nodo raíz, los componentes con múltiples nodos raíz no tienen un comportamiento automático de atributos perdidos. Si `$attrs` no están vinculados explícitamente, se emitirá una advertencia en tiempo de ejecución.

```vue-html
<CustomLayout id="custom-layout" @click="changeValue" />
```

Si `<CustomLayout>` tiene la siguiente plantilla multirraíz, habrá una advertencia porque Vue no puede estar seguro de dónde aplicar los atributos perdidos:

```vue-html
<header>...</header>
<main>...</main>
<footer>...</footer>
```

La advertencia se suprimirá si `$attrs` está vinculado explícitamente:

```vue-html{2}
<header>...</header>
<main v-bind="$attrs">...</main>
<footer>...</footer>
```

## Acceder a los atributos perdidos en JavaScript

<div class="composition-api">

Si es necesario, se puede acceder a los atributos perdidos de un componente en `<script setup>` utilizando la API `useAttrs()`:

```vue
<script setup>
import { useAttrs } from 'vue'

const attrs = useAttrs()
</script>
```

Si no se usa `<script setup>`, `attrs` se declarará como una propiedad del contexto `setup()`:

```js
export default {
  setup(props, ctx) {
    // los atributos perdidos se exponen como ctx.attrs
    console.log(ctx.attrs)
  }
}
```

Note that although the `attrs` object here always reflect the latest fallthrough attributes, it isn't reactive (for performance reasons). You cannot use watchers to observe its changes. If you need reactivity, use a prop. Alternatively, you can use `onUpdated()` to perform side effects with latest `attrs` on each update.
Hay que tener en cuenta que aunque el objeto `attrs` aquí siempre refleja los últimos atributos perdidos, no es reactivo (por razones de rendimiento). No se puede utilizar observadores (watchers) para seguir sus cambios. Si necesitamos reactividad, podemos usar un prop. Alternativamente, se puede usar `onUpdated()` para realizar efectos secundarios con los últimos `attrs` en cada actualización.

</div>

<div class="options-api">

Si fuese necesario, se puede acceder a los atributos perdidos de un componente a través de la propiedad de instancia `$attrs`:

```js
export default {
  created() {
    console.log(this.$attrs)
  }
}
```

</div>
