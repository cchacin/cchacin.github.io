---
layout: post
title: Inyección de Dependencias en Java
description: Este artículo, describe el concepto de Inyección de dependencias en Java y cómo nos ayuda a tener una base de código más modular y desacoplada
tags: java dependency-injection di best-practices
image: /public/images/dependency-injection-in-java/Collaborators.png
---

![Collaborators](https://cchacin.github.io/public/images/dependency-injection-in-java/Collaborators.png)

> Read the English Version [here](https://cchacin.github.io/2019/11/14/dependency-injection-in-java/)
>
> Éste artículo fue originalmente publicado en [cchacin.github.io](https://cchacin.github.io/2020/02/10/dependency-injection-in-java-spanish/)

Java es un lenguaje orientado a objetos con algunos aspectos funcionales incluidos en su núcleo. Al igual que cualquier otro lenguaje orientado a objetos, las clases y los objetos son la base de cualquier funcionalidad que podamos escribir y usar. Las relaciones entre las clases / objetos permiten ampliar y reutilizar la funcionalidad. Sin embargo, la forma en que elegimos construir esas relaciones determina cuán modular, desacoplada y reutilizable es nuestra base de código, no solo en términos de nuestro código de producción sino también en nuestras suites de prueba.

En este artículo, vamos a describir el concepto de Inyección de dependencias en Java y cómo nos ayuda a tener una base de código más modular y desacoplada, lo que nos facilita la vida, incluso para las pruebas, sin la necesidad de ningún contenedor o marco sofisticado.

<!-- more -->

## ¿Qué es una Dependencia?

Cuando una clase `ClassA` usa cualquier método de otra clase` ClassB`, podemos decir que `ClassB` es una dependencia de` ClassA`.

![1](https://cchacin.github.io/public/images/dependency-injection-in-java/1.png)

```java
class ClassA {

  ClassB classB = new ClassB();

  int tenPercent() {
    return classB.calculate() * 0.1d;
  }
}
```

En este ejemplo, `ClassA` está calculando el 10% del valor, y calculando ese valor, está reutilizando la funcionalidad expuesta por` ClassB`.

![2](https://cchacin.github.io/public/images/dependency-injection-in-java/2.png)

Y se puede usar así:

```java
class Main {
  public static void main(String... args) {
    ClassA classA = new ClassA();

    System.out.println("Ten Percent: " + classA.tenPercent());
  }
}
```

Ahora, hay un gran problema con este enfoque:

> `ClassA` **está estrechamente acoplada con** `ClassB`

Si necesitamos cambiar / reemplazar `ClassB` con` ClassC` porque `ClassC` tiene una versión optimizada del método` Calculate () `, necesitamos recompilar` ClassA` porque no tenemos una manera de cambiar esa dependencia , está codificado dentro de `ClassA`.

![Collision](https://cchacin.github.io/public/images/dependency-injection-in-java/Collision.png)

## El Principio de Inyección de Dependencias

El **Principio de inyección de dependencia** no es más que poder pasar (`inyectar`) las dependencias cuando sea necesario en lugar de inicializar las dependencias dentro de la clase receptora.

> Desacoplar la construcción de sus clases de la construcción de las dependencias de sus clases.

## Tipos de Inyección de Dependencias en Java

### Inyección con Setters (No recomendado)

```java
class ClassA {

  ClassB classB;

  /* Setter Injection */
  void setClassB(ClassB injected) {
    classB = injected;
  }

  int tenPercent() {
    return classB.calculate() * 0.1d;
  }
}
```

Con este enfoque, eliminamos la palabra clave `new` de nuestra` ClassA`. Por lo tanto, alejamos la responsabilidad de la creación de `ClassB` de` ClassA`.

`ClassA` todavía tiene una fuerte dependencia de` ClassB` pero ahora se puede `inyectar` desde el exterior:

```java
class Main {
  public static void main(String... args) {
    ClassA classA = new ClassA();
    ClassB classB = new ClassB();

    classA.setClassB(classB);

    System.out.println("Ten Percent: " + classA.tenPercent());
  }
}
```

El ejemplo anterior es mejor que el enfoque inicial porque ahora podemos `inyectar` en` ClassA` una instancia de `ClassB` o incluso mejor, una subclase de` ClassB`:

```java
class ImprovedClassB extends ClassB {
  // content omitted
}
```

```java
class Main {
  public static void main(String... args) {
    ClassA classA = new ClassA();
    ImprovedClassB improvedClassB = new ImprovedClassB();

    classA.setClassB(improvedClassB);

    System.out.println("Ten Percent: " + classA.tenPercent());
  }
}
```

Pero hay un problema significativo con el enfoque de `Inyección con Setters`:

Estamos ocultando la dependencia `ClassB` en` ClassA` porque al leer la firma del constructor, no podemos identificar sus dependencias de inmediato. El siguiente código provoca una `NullPointerException` en tiempo de ejecución:

```java
class Main {
  public static void main(String... args) {
    ClassA classA = new ClassA();

    System.out.println("Ten Percent: " + classA.tenPercent()); // NullPointerException here
  }
}
```

![npe](https://cchacin.github.io/public/images/dependency-injection-in-java/npe.png)

En un lenguaje de tipo estático como Java, siempre es bueno dejar que el compilador nos ayude. Ver `Inyección con Constructor`


### Inyección con Constructor (Altamente recomendado)

```java
class ClassA {

  ClassB classB;

  /* Constructor Injection */
  ClassA(ClassB injected) {
    classB = injected;
  }

  int tenPercent() {
    return classB.calculate() * 0.1d;
  }
}
```

`ClassA` todavía tiene una fuerte dependencia de` ClassB` pero ahora se puede `inyectar` desde afuera usando el constructor:

```java
class Main {
  public static void main(String... args) {
    /* Notice that we are creating ClassB fisrt */
    ClassB classB = new ImprovedClassB();

    /* Constructor Injection */
    ClassA classA = new ClassA(classB);

    System.out.println("Ten Percent: " + classA.tenPercent());
  }
}
```

**VENTAJAS:**
- La funcionalidad permanece intacta en comparación con el enfoque de `Inyección Setter`
- Eliminamos la inicialización `nueva` de la` ClaseA`.
- Todavía podemos inyectar una subclase especializada de `ClassB` a` ClassA`.
- Ahora el compilador nos pedirá las dependencias que necesitamos en tiempo de compilación.

![Happy](https://cchacin.github.io/public/images/dependency-injection-in-java/Happy.png)

### Inyección con Fields (Niños no intenten esto en casa)

Hay una tercera forma de inyectar dependencias en Java, y se llama `Inyección con Fields`. La única manera de que funcione la inyección de campo es:

- Mutar el campo porque es un campo no privado y no final
- Mutar un campo final / privado usando la reflexión

Este enfoque tiene los mismos problemas expuestos por el enfoque de `Inyección con Setters` y además agrega complejidad debido a la mutación / reflexión requerida. Desafortunadamente, este es un patrón bastante común cuando las personas usan un `Framework de inyección de dependencias`.

NOTA:

> Cuando una clase `ClassA` usa cualquier método de otra clase` ClassB` podemos decir que `ClassB` es una dependencia de` ClassA`.

> Si `ClassA` tiene una dependencia de` ClassB`, el constructor `ClassA` debería requerir` ClassB`.

![Feedback](https://cchacin.github.io/public/images/dependency-injection-in-java/Feedback.png)

## Ejemplo Realista

Cada ejemplo de `Hello World` para cualquier idea, concepto o patrón es muy simple de entender y simplemente funciona bien. Pero cuando necesitamos implementarlo en un proyecto real, las cosas se vuelven más complicadas y, a menudo, como ingenieros, tendemos a tratar de resolver el problema introduciendo nuevas capas al problema en lugar de comprender cuál es el problema real.

Ahora que conocemos las ventajas del `Principio de Inyección de Dependencia` usando el enfoque de `Inyección de constructor`, creemos un ejemplo más realista para ver algunos inconvenientes y cómo podemos resolverlo sin introducir una nueva capa en la mezcla.

### La Aplicación de Tareas (ToDo's App)
![Todos](https://cchacin.github.io/public/images/dependency-injection-in-java/Todos.png)

Diseñemos una aplicación de Todo para realizar operaciones CRUD (Crear, Leer, Actualizar, Eliminar) para administrar nuestra lista de tareas, y una arquitectura original puede ser así:

![3](https://cchacin.github.io/public/images/dependency-injection-in-java/3.png)

- `TodoApp` es la clase principal que va a inicializar nuestra aplicación; Puede ser una aplicación de Android, una página web o una aplicación de escritorio con cualquier marco.
- `TodoView` es la clase que mostraría una vista para interactuar, esta clase va a delegar los aspectos relacionados con los datos al` TodoHttpClient`. Su única responsabilidad es pintar / dibujar / renderizar la información y obtener la entrada para realizar acciones contra los datos utilizando la dependencia `TodoHttpClient`.
- `TodoHttpClient` es la clase que contiene un conjunto de métodos HTTP para persistir objetos` Todo` utilizando una API REST.
- `Todo` es un objeto de valor que representa un elemento de todo en nuestro almacén de datos.

<!-- <img src="http://yuml.me/diagram/scruffy/class/[TodoApp]->[TodoView],[TodoView]=>[TodoProvider],[TodoHttpClient]->[TodoProvider],[TodoApp]=>[TodoHttpClient]" alt="todoApp" /> -->

![4](https://cchacin.github.io/public/images/dependency-injection-in-java/4.png)

Escribamos las clases de Java para nuestro diseño usando el enfoque de `Inyección del constructor` que acabamos de aprender:

```java
class Todo {
  /* Value Object class */
  // content omitted
}
```

```java
class TodoApp {
  private final TodoView todoView;

  TodoApp(final TodoView todoView) {
    this.todoView = todoView;
  }
  // content omitted
}
```

```java
class TodoView {
  private final TodoHttpClient todoHttpClient;

  TodoView(final TodoHttpClient todoHttpClient) {
    this.todoHttpClient = todoHttpClient;
  }
  // content omitted
}
```

```java
class Main {
  public static void main(String... args) {
    new TodoApp(new TodoView(new TodoHttpClient("https://api.todos.io/")));
  }
}
```

Ahora enfoquemos nuestra atención en la relación entre las clases `TodoView` y` TodoHttpClient` y agreguemos más detalles:

![Magic](https://cchacin.github.io/public/images/dependency-injection-in-java/Magic.png)

```java
class TodoHttpClient extends MyMagicalHttpAbstraction {

  TodoView(final String baseUrl) {
    super(baseUrl);
  }

  @GET
  List<Todo> getAll() {
    return super.get(Todo.class);
  }

  @GET
  Todo get(long id) {
    return super.get(Todo.class, id);
  }

  @POST
  long save(Todo todo) {
    return super.post(todo);
  }

  @PUT
  Todo update(Todo todo) {
    return super.put(todo, todo.getId());
  }

  @DELETE
  void delete(long id) {
    super.delete(Todo.class, id);
  }
}
```

```java
class TodoView extends MyFrameworkView {
  private final TodoHttpClient httpClient;

  // View initialized by the view library/framework
  // or injected as a dependency as well
  private ListView listView;
  private DetailView detailView;

  TodoView(final TodoHttpClient httpClient) {
    this.httpClient = httpClient;
  }

  void showTodos() {
    listView.add(httpClient.getAll());
  }

  void showTodo(Todo selected) {
    detailView.print(httpClient.get(selected.getId()));
  }

  void save(Todo todo) {
    httpClient.save(todo);
    listView.add(todo)
  }

  void update(Todo todo) {
    httpClient.update(todo);
    detailView.refresh(todo);
  }

  void delete(long id) {
    httpClient.delete(id);
    listView.refresh();
  }
}
```

## Testing our design
![Scientist](https://cchacin.github.io/public/images/dependency-injection-in-java/Scientist.png)

Creemos una prueba unitaria para la clase `TodoView` donde probamos la clase de forma aislada sin instanciar ninguna de sus dependencias. En este caso, la dependencia es `TodoHttpClient`:

```java
@ExtendWith(MockitoExtension.class)
class TodoViewTest {

  @Test
  void shouldBeEmptyWhenEmptyList(@Mock TodoHttpClient httpClient) {
    // Given
    Mockito.when(httpClient.getAll()).thenReturn(List.of());

    // When
    TodoView todoView = new TodoView(httpClient);
    todoView.showTodos();

    // Then
    Assertions.assertThat(todoView.getListView()).isEmpty();
  }
}
```

Ahora que tenemos nuestro caso de prueba aprobado, analicemos cómo nuestro diseño impacta el enfoque de prueba:

- Incluimos la librería [Mockito](https://site.mockito.org/) para poder crear una instancia falsa de `TodoHttpClient`, y eso agrega mucha complejidad.
- Tenemos que preparar nuestra instancia de `TodoHttpClient` para falsificar el regreso de una lista vacía al llamar al método` getAll () `, ahora nuestra prueba de unidad también contiene detalles de implementación sobre el` TodoHttpClient`.
- Además, dado que `TodoHttpClient` es una clase concreta, no podemos cambiar la implementación para llamar a un DB sin tener que cambiar también la clase` TodoView`, y tendríamos que reescribir las pruebas unitarias incluso cuando deberían aislar esta implementación detalle.

## Mejoremos nuestro diseño

![VideoLearning](https://cchacin.github.io/public/images/dependency-injection-in-java/VideoLearning.png)

Una cosa que podemos hacer para desacoplar nuestras clases es introducir una interfaz, ya que el lenguaje Java siempre es bueno confiar en abstracciones en lugar de confiar en implementaciones reales.

Pongamos una interfaz entre `TodoView` y` TodoHttpClient`:

![5](https://cchacin.github.io/public/images/dependency-injection-in-java/5.png)

**TodoProvider**

```java
interface TodoProvider {
  List<Todo> getAll();
  Todo get(long id);
  long save(Todo todo);
  Todo update(Todo todo);
  void delete(long id);
}
```

Hagamos el `TodoHttpClient` para implementar esa interfaz:

```java
class TodoHttpClient extends MyMagicalHttpAbstraction implements TodoProvider {

  TodoView(final String baseUrl) {
    super(baseUrl);
  }

  @GET
  List<Todo> getAll() {
    return super.get(Todo.class);
  }

  @GET
  Todo get(long id) {
    return super.get(Todo.class, id);
  }

  @POST
  long save(Todo todo) {
    return super.post(todo);
  }

  @PUT
  Todo update(Todo todo) {
    return super.put(todo, todo.getId());
  }

  @DELETE
  void delete(long id) {
    super.delete(Todo.class, id);
  }
}
```

Ahora la clase `TodoView` se ve así:

```java
class TodoView extends MyFrameworkView {
  private final TodoProvider provider;

  // View initialized by the view library/framework
  // or injected as a dependency as well
  private ListView listView;
  private DetailView detailView;

  TodoView(final TodoProvider httpClient) {
    this.provider = provider;
  }

  void showTodos() {
    listView.add(provider.getAll());
  }

  void showTodo(Todo selected) {
    detailView.print(provider.get(selected.getId()));
  }

  void save(Todo todo) {
    provider.save(todo);
    listView.add(todo)
  }

  void update(Todo todo) {
    provider.update(todo);
    detailView.refresh(todo);
  }

  void delete(long id) {
    provider.delete(id);
    listView.refresh();
  }
}
```

**¿Qué ganamos con estos cambios?**

Podemos cambiar el `TodoHttpClient` con algo como` TodoDBProvider` en el `TodoApp` y el comportamiento de la aplicación seguirá siendo el mismo:

```java
new TodoApp(new TodoView(new TodoDbProvider("dbName", "dbUser", "dbPassword")));
```

## Veamos cómo eso ayuda en las pruebas unitarias
![Library](https://cchacin.github.io/public/images/dependency-injection-in-java/Library.png)

```java
@ExtendWith(MockitoExtension.class)
class TodoViewTest {

  @Test
  void shouldBeEmptyWhenEmptyList(@Mock TodoProvider provider) {
    // Given
    Mockito.when(provider.getAll()).thenReturn(List.of());

    // When
    TodoView todoView = new TodoView(httpClient);
    todoView.showTodos();

    // Then
    Assertions.assertThat(todoView.getListView()).isEmpty();
  }
}
```

La prueba sigue pasando (verde), lo cual es genial, pero espera ... nada cambió en realidad :(

Los únicos cambios estaban relacionados con los nombres:

- `TodoHttpClient` ->` TodoProvider` no tiene valor para la prueba.
- `httpClient` ->` provider` no tiene valor para la prueba aquí.
- Todavía confiamos en la librería Mockito.
- Todavía estamos acoplando la prueba al nombre de la interfaz: `TodoProvider`.
- Todavía estamos acoplando la prueba al nombre del método: `getAll ()`

## ¿Podemos eliminar Mockito?

Si ahora tenemos una interfaz, ¿por qué estamos acoplados a Mockito para crear un objeto falso que podemos crear manualmente usando una clase anónima? Cambiemos eso:

```java
@ExtendWith(MockitoExtension.class)
class TodoViewTest {

  // Given
  TodoProvider provider = new TodoProvider() {
    @Override
    public List<TodoItem> getAll() {
      return List.of();
    }

    @Override
    public TodoItem get(long id) {
      return null;
    }

    @Override
    public long save(TodoItem todo) {
      return 0;
    }

    @Override
    public TodoItem update(TodoItem todo) {
      return null;
    }

    @Override
    public void delete(long id) {

    }
  };

  // When
  var todoView = new TodoView(provider);
  todoView.displayListView();

  // Then
  assertThat(todoView.getTodoItemList()).isEmpty();
}
```

Genial, ahora nuestro diseño es más flexible, ya que podemos inyectar una implementación diferente de `TodoProvider`, y podemos hacer lo mismo en nuestras pruebas sin usar Mockito. Pero estamos pagando el precio: **Verbosidad**, Mockito elimina la necesidad de implementar todos los métodos de las interfaces.

## Solo el comienzo

En el siguiente artículo, eliminemos la verbosidad de las pruebas y escribamos un diseño aún mejor.

Estén atentos para más publicaciones como esta.
