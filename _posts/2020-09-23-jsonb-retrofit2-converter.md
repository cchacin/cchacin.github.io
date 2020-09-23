---
layout: post
title:  🌒 JakartaEE JSON-B 🐝 Retrofit2 Converter
description: A Retrofit2 Converter.Factory for JakartaEE JSON Binding
lang: en-us
author: cchacin
tags: jakartaee jsonb serialization retrofit
---

![java-records-jsonb](/public/images/retrofit2.png)

> Retrofit is pluggable allowing different serialization formats and their libraries to be used for converting Java types to their HTTP representation and parsing HTTP entities back into Java types.
>
> These are called converters, and Retrofit includes a few first-party modules for popular frameworks

Just for fun, I created a [Retrofit2][2] `Converter.Factory` for [JakartaEE Json-B][1].

<!-- more -->

### Usage

#### Add the dependencies:

##### Retrofit2

```xml
<dependency>
  <groupId>com.squareup.retrofit2</groupId>
  <artifactId>retrofit</artifactId>
  <version>2.9.0</version>
</dependency>
```

##### OkHttp3

```xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.14.9</version>
</dependency>
```

##### JakartaEE Json Binding Reference Implementation

```xml
<dependency>
  <groupId>org.eclipse</groupId>
  <artifactId>yasson</artifactId>
  <version>1.0.8</version>
</dependency>
```


##### JsonB Retrofit2 Converter

```xml
<dependency>
  <groupId>io.github.cchacin</groupId>
  <artifactId>jsonb-retrofit-converter</artifactId>
  <version>1.0.2</version>
</dependency>
```

Add a converter factory when building your `Retrofit` instance using the
`create` method:

```java
var retrofit = new Retrofit.Builder()
    .baseUrl("https://example.com/")
    .addConverterFactory(JsonbConverterFactory.create())
    .build();
```

Alternatively, you can pass an instance of `javax.json.bind.Jsonb`:

```java
var jsonb = JsonbBuilder.create(/* configure your jsonb instance here */ );
var retrofit = new Retrofit.Builder()
    .baseUrl("https://example.com/")
    .addConverterFactory(JsonbConverterFactory.create(jsonb))
    .build();
```

Now we are able to use **JakartaEE JsonB** to serialize and deserialize POJOs using **Retrofit2**

The code is available on [Github][3].

[1]: (http://json-b.net/)
[2]: (https://square.github.io/retrofit/)
[3]: (https://github.com/cchacin/jsonb-retrofit-converter/)
