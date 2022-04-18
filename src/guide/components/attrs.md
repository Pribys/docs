---
outline: deep
---

# Atributos fallidos (Fallthrough)

> Se supone que ya conoces lo tratado en [lo básico sobre componentes](/guide/essentials/component-basics). Si eres novato, deberías leer primero esa sección.

## Herencia de atributos

Un "atributo fallido" (o "fallthrough attribute" en inglés) es un atributo u oyente `v-on` que se pasa a un componente, pero que no se declara explícitamente en los [props](./props) o [emits](./events.html#declarar-los-eventos-emitidos) del componente receptor. Los ejemplos comunes de esto incluyen los atributos `class`, `estyle` e `id`.

Cuando un componente representa un único elemento raíz, los atributos fallidos se agregarán automáticamente a los atributos del elemento raíz. Por ejemplo, dado un componente `<MiBoton>` con la siguiente plantilla:

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

Entonces, los atributos fallidos recibidos por `<MiBoton>` se reenviarán automáticamente a `<BotonBase>`.

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

The common scenario for disabling attribute inheritance is when attributes need to be applied to other elements besides the root node. By setting the `inheritAttrs` option to `false`, you can take full control over where the fallthrough attributes should be applied.
El escenario común para deshabilitar la herencia de atributos es cuando los atributos deben aplicarse a otros elementos además del nodo raíz. Al establecer la opción `inheritAttrs` en `false`, podemos tomar el control total sobre dónde se deben aplicar los atributos fallidos.

Se puede acceder a estos atributos fallidos directamente en expresiones de plantilla como `$attrs`:

```vue-html
<span>Atributos fallidos: {{ $attrs }}</span>
```

El objeto `$attrs` incluye todos los atributos que no están declarados por las opciones `props` o `emits` del componente (por ejemplo, `class`, `style`, oyentes `v-on`, etc.).

Algunas notas:
  
- A diferencia de los props, los atributos fallidos conservan su escritura original en JavaScript, por lo que se debe acceder a un atributo como `foo-bar` como `$attrs['foo-bar']`.

- Un oyente de eventos `v-on` como `@click` se incluirá en el objeto como una función en `$attrs.onClick`.

Usando nuestro ejemplo de componente `<MiBoton>` de la [sección anterior](#attribute-inheritance), a veces es posible que necesitemos envolver el elemento `<button>` real con un `<div>` adicional por motivos de estilo:

```vue-html
<div class="btn-wrapper">
  <button class="btn">pulsar</button>
</div>
```

We want all fallthrough attributes like `class` and `v-on` listeners to be applied to the inner `<button>`, not the outer `<div>`. We can achieve this with `inheritAttrs: false` and `v-bind="$attrs"`:

```vue-html{2}
<div class="btn-wrapper">
  <button class="btn" v-bind="$attrs">click me</button>
</div>
```

Remember that [`v-bind` without an argument](/guide/essentials/template-syntax.html#dynamically-binding-multiple-attributes) binds all the properties of an object as attributes of the target element.

## Attribute Inheritance on Multiple Root Nodes

Unlike components with a single root node, components with multiple root nodes do not have an automatic attribute fallthrough behavior. If `$attrs` are not bound explicitly, a runtime warning will be issued.

```vue-html
<CustomLayout id="custom-layout" @click="changeValue" />
```

If `<CustomLayout>` has the following multi-root template, there will be a warning because Vue cannot be sure where to apply the fallthrough attributes:

```vue-html
<header>...</header>
<main>...</main>
<footer>...</footer>
```

The warning will be suppressed if `$attrs` is explicitly bound:

```vue-html{2}
<header>...</header>
<main v-bind="$attrs">...</main>
<footer>...</footer>
```

## Accessing Fallthrough Attributes in JavaScript

<div class="composition-api">

If needed, you can access a component's fallthrough attributes in `<script setup>` using the `useAttrs()` API:

```vue
<script setup>
import { useAttrs } from 'vue'

const attrs = useAttrs()
</script>
```

If not using `<script setup>`, `attrs` will be exposed as a property of the `setup()` context:

```js
export default {
  setup(props, ctx) {
    // fallthrough attributes are exposed as ctx.attrs
    console.log(ctx.attrs)
  }
}
```

Note that although the `attrs` object here always reflect the latest fallthrough attributes, it isn't reactive (for performance reasons). You cannot use watchers to observe its changes. If you need reactivity, use a prop. Alternatively, you can use `onUpdated()` to perform side effects with latest `attrs` on each update.

</div>

<div class="options-api">

If needed, you can access a component's fallthrough attributes via the `$attrs` instance property:

```js
export default {
  created() {
    console.log(this.$attrs)
  }
}
```

</div>
