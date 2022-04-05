# Un Curso Acelerado

¡Vamos a saltar directamente a eso! Aprendamos Vue Test Utils (VTU) creando una aplicación Todo simple y escribiendo pruebas sobre la marcha. Esta guía cubrirá cómo:

- Componentes de montaje
- Buscar elementos
- Llenar formularios
- Activar eventos

## Empezando

Comenzaremos con un componente `TodoApp` simple con una sola tarea pendiente:

```vue
<template>
  <div></div>
</template>

<script>
export default {
  name: 'TodoApp',

  data() {
    return {
      todos: [
        {
          id: 1,
          text: 'Learn Vue.js 3',
          completed: false
        }
      ]
    }
  }
}
</script>
```
## La primera prueba - se procesa una tarea pendiente

La primera prueba que escribiremos verifica que se renderice una tarea pendiente. Veamos primero la prueba, luego analicemos cada parte:

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('renders a todo', () => {
  const wrapper = mount(TodoApp)

  const todo = wrapper.get('[data-test="todo"]')

  expect(todo.text()).toBe('Learn Vue.js 3')
})
```
Comenzamos importando el montaje: esta es la forma principal de representar un componente en VTU. Declara una prueba utilizando la función de prueba con una breve descripción de la prueba. Las funciones `test` y `expect` están disponibles globalmente en la mayoría de los ejecutores de pruebas (este ejemplo usa [Vitest](https://vitest.dev/)). Si `test` y `expect` parece confuso, [aquí](../comenzar/tdd.html) encontraras lo que debes conocer antes de probar componentes de Vue.

A continuación, llamamos a `mount` y pasamos el componente como primer argumento; esto es algo que casi todas las pruebas (de componentes Vue) que escribas harán. Por convención, asignamos el resultado a una variable llamada `wrapper`, ya que `mount` proporciona un "envoltorio" simple alrededor de la aplicación con algunos métodos convenientes para realizar pruebas.

Finalmente, usamos otra función global común a muchos ejecutores de pruebas - incluido Vitest - `expect`. La idea es que estamos afirmando, o _esperando_, que el resultado real coincida con lo que creemos que debería ser. En este caso, estamos encontrando un elemento con el selector `data-test="todo"` - en el DOM, se verá como `<div data-test="todo">...</div>`. Luego llamamos al método `text` para obtener el contenido, que esperamos sea `'Learn Vue.js 3'`.


>No es requerido usar selectores de `data-test`, pero puede hacer que sus pruebas sean menos frágiles. Las clases y los identificadores tienden a cambiar o moverse a medida que crece la aplicación - al usar `data-test`, queda claro para otros desarrolladores qué elementos se usan en las pruebas y no deben cambiarse.

## Haciendo pasar la prueba

Si ejecutamos esta prueba ahora, falla con el siguiente mensaje de error: `Unable to get [data-test="todo"]`. Esto se debe a que no estamos procesando ningún elemento pendiente, por lo que la llamada `get()` no puede devolver un contenedor (recuerde, VTU envuelve todos los componentes y elementos DOM en un "envoltorio" con algunos métodos convenientes). Actualicemos `<template>` en `TodoApp.vue` para renderizar el arreglo `todos`:

```vue
<template>
  <div>
    <div v-for="todo in todos" :key="todo.id" data-test="todo">
      {{ todo.text }}
    </div>
  </div>
</template>
```
Con este cambio, la prueba está pasando. ¡Felicidades! Escribiste tu primera prueba de componentes.

## Adding a new todo

Límite

```vue
<template>
  <div>
    <div
      v-for="todo in todos"
      :key="todo.id"
      data-test="todo"
      :class="[todo.completed ? 'completed' : '']"
    >
      {{ todo.text }}
      <input
        type="checkbox"
        v-model="todo.completed"
        data-test="todo-checkbox"
      />
    </div>

    <form data-test="form" @submit.prevent="createTodo">
      <input data-test="new-todo" v-model="newTodo" />
    </form>
  </div>
</template>

<script>
export default {
  name: 'TodoApp',
  data() {
    return {
      newTodo: '',
      todos: [
        {
          id: 1,
          text: 'Learn Vue.js 3',
          completed: false
        }
      ]
    }
  },
  methods: {
    createTodo() {
      this.todos.push({
        id: 2,
        text: this.newTodo,
        completed: false
      })
    }
  }
}
</script>
```

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('renders a todo', () => {
  const wrapper = mount(TodoApp)

  const todo = wrapper.get('[data-test="todo"]')

  expect(todo.text()).toBe('Learn Vue.js 3')
  
  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(1)
})

test('creates a todo', async () => {
  const wrapper = mount(TodoApp)
  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(1)

  await wrapper.get('[data-test="new-todo"]').setValue('New todo')
  await wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})

test('completes a todo', async () => {
  const wrapper = mount(TodoApp)
  
  await wrapper.get('[data-test="todo-checkbox"]').setValue(true)

  expect(wrapper.get('[data-test="todo"]').classes()).toContain('completed')
})
```



