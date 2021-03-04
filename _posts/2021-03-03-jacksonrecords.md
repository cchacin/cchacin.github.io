---
layout: post
date: "2021-03-03 17:33:16 -0800"
title:  💾 Java Records 💿 with Jackson 2.12
description: Java Record is a new kind of type declaration in the Java language. In this article, we are going to see how we can serialize and deserialize records to/from JSON using Jackson 2.12.0.
lang: en-us
author: cchacin
tags: jackson java14 records serialization
---

![java-records-jackson](https://carloschac.in/public/images/java-records-jackson.png)

In the previous article about Java 14 Records, we saw how to start creating Records to avoid writing much boilerplate code that the compiler would generate for us.

Now the next steps are to see how we can serialize records to JSON and deserialize JSON to records to be able to use them as a request/response representation for microservices.

In this case, we would use the Jackson 2.12+.

<!-- more -->

Continuing with the [same example](https://carloschac.in/2020/04/17/java-records/) that we used in the previous article, we would need to add Jackson Dependencies to our existing pom.xml file:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.0</version>
</dependency>
```

### 💾 Now let's see our example Record:

```java
record Person(
    @JsonProperty("first_name") String firstName,
    @JsonProperty("last_name") String lastName,
    String address,
    Date birthday,
    List<String> achievements) {
}
```

### 💡 Java 14+ compiler would generate all of the following:

```bash
$ javap -p Person.class
```

```java
final class Person extends java.lang.Record {
  private final java.lang.String firstName;
  private final java.lang.String lastName;
  private final java.lang.String address;
  private final java.util.Date birthday;
  private final java.util.List<java.lang.String> achievements;
  public Person(
    java.lang.String,
    java.lang.String,
    java.lang.String,
    java.util.Date,
    java.util.List<java.lang.String>);
  public final java.lang.String toString();
  public final int hashCode();
  public final boolean equals(java.lang.Object);
  public java.lang.String firstName();
  public java.lang.String lastName();
  public java.lang.String address();
  public java.util.Date birthday();
  public java.util.List<java.lang.String> achievements();
}
```

### 🔨 Our test setup:

```java
final ObjectMapper mapper = new ObjectMapper()
      .enable(SerializationFeature.INDENT_OUTPUT);
```

To serialize/deserialize a record like this:

```java
var person = new Person(
    "John",
    "Doe",
    "USA",
    new Date(981291289182L),
    List.of("Speaker")
);
```

To/From a json string like this:

```json
{
  "first_name" : "John",
  "last_name" : "Doe",
  "address" : "USA",
  "birthday" : 981291289182,
  "achievements" : [ "Speaker" ]
}
```

### 💊 Testing Serialization

```java
@Test
void serializeRecord() throws Exception {
    // Given
    var person = new Person(
            "John",
            "Doe",
            "USA",
            new Date(981291289182L),
            List.of("Speaker")
    );

    var json = """
            {
              "first_name" : "John",
              "last_name" : "Doe",
              "address" : "USA",
              "birthday" : 981291289182,
              "achievements" : [ "Speaker" ]
            }""";

    // When
    var serialized = mapper.writeValueAsString(person);

    // Then
    assertThat(serialized).isEqualTo(json);
}
```

## 💊 Testing Deserialization

Let's use the same record to try the deserialization using also the same configuration:

```java
@Test
void deserializeRecord() throws Exception {
    // Given
    var person = new Person(
            "John",
            "Doe",
            "USA",
            new Date(981291289182L),
            List.of("Speaker")
    );

    var json = """
            {
              "first_name" : "John",
              "last_name" : "Doe",
              "address" : "USA",
              "birthday" : 981291289182,
              "achievements" : [ "Speaker" ]
            }""";

    // When
    var deserialized = mapper.readValue(json, Person.class);

    // Then
    assertThat(deserialized).isEqualTo(person);
}
```

## 🔆 Conclusions
- ✅ We can start using Jackson >= [2.12.0](https://github.com/FasterXML/jackson/wiki/Jackson-Release-2.12) to serialize/deserialize Java Records.
- 👷 This is a new feature, and there can be edge cases, try it out with caution.
- 🐞 Report issues [here](https://github.com/FasterXML/jackson-core/issues) if you find any
