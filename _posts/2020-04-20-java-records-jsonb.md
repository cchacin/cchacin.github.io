---
layout: post
title:  💾 Java 14 Records 🐞 with JakartaEE JSON-B
description: Java 14 Record is a new kind of type declaration in the Java language. In this article, we are going to see how we can serialize and deserialize records to/from JSON using JakartaEE JSON-B.
lang: en-us
tags: jakartaee jsonb java14 records serialization
cover_image: https://carloschac.in/public/images/java-records-jsonb.png
---

![java-records-jsonb](https://carloschac.in/public/images/java-records-jsonb.png)

> The article was initially published at [carloschac.in](https://carloschac.in/2020/04/20/java-records-jsonb/)

## 💾 Java 14 Records 🐞 with JakartaEE JSON-B

In the previous article about Java 14 Records, we saw how to start creating Records to avoid writing much boilerplate code that the compiler would generate for us.

Now the next steps are to see how we can serialize records to JSON and deserialize JSON to records to be able to use them as a request/response representation for microservices.

In this case, we would use the JSON-B specification used by JakartaEE and MicroProfile implementations like GlashFish, TomEE, Wildfly, OpenLiberty, and Quarkus.

<!-- more -->

Continuing with the [same example](https://carloschac.in/2020/04/17/java-records/) that we used in the previous article, we would need to add a JSON-B implementation, let's add the Eclipse Yasson dependency to our existing pom.xml file:

```xml
<dependency>
    <groupId>org.eclipse</groupId>
    <artifactId>yasson</artifactId>
    <version>1.0.6</version>
    <scope>test</scope>
</dependency>
```

### 💾 Now let's see our example Record:

```java
record Person(
    String firstName,
    String lastName,
    String address,
    LocalDate birthday,
    List<String> achievements) {
}
```

### 💡 Java 14+ compiler would generate all of the following:

```bash
$ javap -p Person.class
```

```java
public final class Person extends java.lang.Record {
  private final java.lang.String firstName;
  private final java.lang.String lastName;
  private final java.lang.String address;
  private final java.time.LocalDate birthday;
  private final java.util.List<java.lang.String> achievements;
  public Person(java.lang.String, java.lang.String, java.lang.String, java.time.LocalDate, java.util.List<java.lang.String>);
  public java.lang.String toString();
  public final int hashCode();
  public final boolean equals(java.lang.Object);
  public java.lang.String firstName();
  public java.lang.String lastName();
  public java.lang.String address();
  public java.time.LocalDate birthday();
  public java.util.List<java.lang.String> achievements();
}
```

### 🔨 Our test case would consist of using Yasson

To serialize a record like this:

```java
var person = new Person(
    "John",
    "Doe",
    "USA",
    LocalDate.of(1990, 11, 11),
    List.of("Speaker")
);
```

To a json output like this:

```json
{
    "achievements": [
        "Speaker"
    ],
    "address": "USA",
    "birthday": "1990-11-11",
    "firstName": "John",
    "lastName": "Doe"
}
```

### 🔎 Let's check the complete test code:

```java
@Test
void serializeRecords() throws Exception {
    // Given
    var person = new Person(
            "John",
            "Doe",
            "USA",
            LocalDate.of(1990, 11, 11),
            List.of("Speaker")
    );

    var expected = """
            {
                "achievements": [
                    "Speaker"
                ],
                "address": "USA",
                "birthday": "1990-11-11",
                "firstName": "John",
                "lastName": "Doe"
            }""";

    // When
    var jsonb = JsonbBuilder.create(new JsonbConfig().withFormatting(true));
    var serialized = jsonb.toJson(person);

    // Then
    assertThat(serialized).isEqualTo(expected);
}
```

After running the test with maven or in the IDE, we get the following assertion's failure:

```
org.opentest4j.AssertionFailedError:
Expecting:
 <"
{
}">
to be equal to:
 <"
{
    "achievements": [
        "Speaker"
    ],
    "address": "USA",
    "birthday": "1990-11-11",
    "firstName": "John",
    "lastName": "Doe"
}">
but was not.
```

JSON-B is serializing the record as an empty object :(

Let's read the specification:

> For a serialization operation, if a matching public getter method exists, the method is called to obtain the value of the property. If a matching getter method with private, protected or defaulted to package-only access exists, then this field is ignored. If no matching getter method exists and the field is public, then the value is obtained directly from the field.

Let's take a look at the field `firstName` on the record as an example:

```bash
$ javap -p Person.class
```

```java
public final class Person extends java.lang.Record {
  private final java.lang.String firstName; // This field is private
  .
  .
  .
  public java.lang.String firstName(); // This property is not a getter, i.e. getFirstName()
```

We are not complaining with the specification, but we can solve this renaming the field defined in the record to use the JavaBean setters convention:

```java
record Person(
    String getFirstName,
    String getLastName,
    String getAddress,
    LocalDate getBirthday,
    List<String> getAchievements) {
}
```

⚠️ The above example can be problematic if we forget about adding the `get` prefix. For primitive `boolean` properties, the prefix for getters is `is` instead of `get`, also easy to forget.

Another alternative is to specify to the JSON-B implementation to serialize using private fields instead of the getters, for that we need to implement the [PropertyVisibilityStrategy](http://json-b.net/docs/api/java.json.bind/javax/json/bind/config/PropertyVisibilityStrategy.html) interface.

> Provides mechanism on how to define customized property visibility strategy.
This strategy can be set via [JsonbConfig](http://json-b.net/docs/api/java.json.bind/javax/json/bind/JsonbConfig.html).

🔧 The interface has two methods:

- `boolean isVisible(Field field)` Responds whether the given field should be considered as the JsonbProperty.
- `boolean isVisible(Method method)` Responds whether the given method should be considered as the JsonbProperty.

To achieve our goal, we want to return `true` to serialize fields and `false` to serialize methods:

```java
var visibilityStrategy = new PropertyVisibilityStrategy() {
    @Override
    public boolean isVisible(Field field) {
        return true;
    }

    @Override
    public boolean isVisible(Method method) {
        return false;
    }
};
```

And then we pass it to the `JsonbConfig` object:

```java
var jsonb = JsonbBuilder.create(new JsonbConfig().withFormatting(true).withPropertyVisibilityStrategy(visibilityStrategy));
```

### ♻️ Now we have a passing test:

![records-jsonb-passing-test](https://carloschac.in/public/images/records-jsonb-passing-test.png)

## 💊 Testing Deserialization

Let's use the same record to try the deserialization using also the same configuration, the only thing that we need to add to our test is the deserialization itself using the `Jsonb` object and the assertion to compare:

```java
@Test
void serializeRecords() throws Exception {
    // Given
    var person = new Person(
            "John",
            "Doe",
            "USA",
            LocalDate.of(1990, 11, 11),
            List.of("Speaker")
    );

    var json = """

            {
                "achievements": [
                    "Speaker"
                ],
                "address": "USA",
                "birthday": "1990-11-11",
                "firstName": "John",
                "lastName": "Doe"
            }""";

    // When
    var visibilityStrategy = new PropertyVisibilityStrategy() {
        @Override
        public boolean isVisible(Field field) {
            return true;
        }

        @Override
        public boolean isVisible(Method method) {
            return false;
        }
    };
    var jsonb = JsonbBuilder.create(
            new JsonbConfig()
                    .withFormatting(true)
                    .withPropertyVisibilityStrategy(visibilityStrategy)
    );
    var serialized = jsonb.toJson(person);
    var deserialized = jsonb.fromJson(json, Person.class);

    // Then
    assertThat(deserialized).isEqualTo(person);
    assertThat(serialized).isEqualTo(json);
}
```

Our test case fails for deserialization with the following error:

```java
javax.json.bind.JsonbException: Cannot create an instance of a class: class records.Person, No default constructor found.
```

📌 Per the specification, we need either to:

- ✅ Define an empty/null constructor
- ✅ Define a constructor or method annotated with `@JsonbCreator`

But how can we do that if the compiler generates the constructor for a record?

### 💾 Java Records 👷 Compact Constructor:

```java
record Person(
    String firstName,
    String lastName,
    String address,
    LocalDate birthday,
    List<String> achievements) {

    public Person {
        if (birthday >= LocalDate.now()) {
            throw new IllegalArgumentException( "Birthday must be < today!");
        }
    }
}
```

This compact constructor is meant to be used only for validation purposes as in the example above and notice that we don't have to repeat the field parameters nor the field initializations, The remaining initialization code is supplied by the compiler.

But how this helps to our deserialization problem, well, as another regular constructor we can add annotations there, let's fix it:

```java
record Person(
    String firstName,
    String lastName,
    String address,
    LocalDate birthday,
    List<String> achievements) {

    @JsonbCreator
    public Person {}
}
```

This is the decompiled bytecode generated by the compiler:

```java
public final class Person extends java.lang.Record {
    private final java.lang.String firstName;
    private final java.lang.String lastName;
    private final java.lang.String address;
    private final java.time.LocalDate birthday;
    private final java.util.List<java.lang.String> achievements;

    @javax.json.bind.annotation.JsonbCreator
    public Person(java.lang.String firstName, java.lang.String lastName, java.lang.String address, java.time.LocalDate birthday, java.util.List<java.lang.String> achievements) { /* compiled code */ }

    public java.lang.String toString() { /* compiled code */ }

    public final int hashCode() { /* compiled code */ }

    public final boolean equals(java.lang.Object o) { /* compiled code */ }

    public java.lang.String firstName() { /* compiled code */ }

    public java.lang.String lastName() { /* compiled code */ }

    public java.lang.String address() { /* compiled code */ }

    public java.time.LocalDate birthday() { /* compiled code */ }

    public java.util.List<java.lang.String> achievements() { /* compiled code */ }
}
```

We can notice the `@JsonbCreator` annotation passed to the generated constructor. After that change, our test suite for serialization and deserialization of records with JSON-B passes.

## 🔆 Conclusions
- ✅ We can use records to serialize and deserialize JSON request/response objects.
- ✅ We described two ways of achieving serialization:
  - ✅ Renaming the fields to use the getter convention.
  - ✅ Adding a custom `PropertyVisibilityStrategy` to serialize using the private fields.
- ✅ We described how to achieve the deserialization using the `@JsonbCreator` annotation.
- 🔴 Most probably, in the following versions of the JSON-B API specification records would be taken in consideration and the strategies described in this article are not going to be required.
