---
layout: post
title: Immutability in Java Made Easy
description: In this article, we describe how to use the Immutables.org library to generate our immutable classes instead of doing manual error-prone implementations.
tags: java best-practices immutability
image: /public/images/HomeOffice.png
---

![HomeOffice](https://cchacin.github.io/public/images/HomeOffice.png)
# :tophat: Immutability in Java :fire: Made Easy

> The article was initially published at [cchacin.github.io](https://cchacin.github.io/2020/04/11/immutables-library/)

In [Effective Java](http://www.amazon.com/exec/obidos/ASIN/0134685997/ref=nosim/javapractices-20), *Joshua Bloch* makes the following recommendation:

> Classes should be immutable unless there's a very good reason to make them mutable... If a class cannot be made immutable, limit its mutability as much as possible.

## :nut_and_bolt: Immutable Objects

> An object is considered immutable if its state cannot change after it is constructed. Maximum reliance on immutable objects is widely accepted as a sound strategy for creating a simple, reliable code. [reference](https://docs.oracle.com/javase/tutorial/essential/concurrency/immutable.html)

> - Immutable objects are constructed once, in a consistent state, and can be safely shared
>   - Will fail if mandatory attributes are missing
>   - Cannot be sneakily modified when passed to other code
> - Immutable objects are naturally thread-safe and can therefore be safely shared among threads
>   - No excessive copying
>   - No excessive synchronization
>- Object definitions are pleasant to write and read
>   - No boilerplate setter and getters
>   - No ugly IDE-generated `hashCode`, `equals` and `toString` methods that end up being stored in source control. [reference](http://immutables.github.io/immutable.html)


### :wrench: Let's convert a mutable object into an immutable one (by hand :raised_hand:):

The following class is what we usually call [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) or [Java Bean](https://en.wikipedia.org/wiki/JavaBeans):

```java
import java.util.Date;
import java.util.List;
import java.util.Objects;

public class OldModel {

    private String fieldA;
    private Date fieldB;
    private Long fieldC;
    private List<String> fieldD;

    public OldModel() {
    }

    public String getFieldA() {
        return fieldA;
    }

    public void setFieldA(String fieldA) {
        this.fieldA = fieldA;
    }

    public Date getFieldB() {
        return fieldB;
    }

    public void setFieldB(Date fieldB) {
        this.fieldB = fieldB;
    }

    public Long getFieldC() {
        return fieldC;
    }

    public void setFieldC(Long fieldC) {
        this.fieldC = fieldC;
    }

    public List<String> getFieldD() {
        return fieldD;
    }

    public void setFieldD(List<String> fieldD) {
        this.fieldD = fieldD;
    }

    @Override
    public String toString() {
        return "OldModel{" +
                "fieldA='" + fieldA + '\'' +
                ", fieldB=" + fieldB +
                ", fieldC=" + fieldC +
                ", fieldD=" + fieldD +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        OldModel oldModel = (OldModel) o;
        return Objects.equals(getFieldA(), oldModel.getFieldA()) &&
                Objects.equals(getFieldB(), oldModel.getFieldB()) &&
                Objects.equals(getFieldC(), oldModel.getFieldC()) &&
                Objects.equals(getFieldD(), oldModel.getFieldD());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getFieldA(), getFieldB(), getFieldC(), getFieldD());
    }
}
```

To convert that to an immutable object, we have to:

#### :red_circle: The class can not be overridden:
  - :closed_lock_with_key: Make the class final.
  - :closed_lock_with_key: Or make the constructor final and use static factory methods.

```diff
-public class OldModel {
+public final class OldModel {
```

#### :closed_lock_with_key: Make all the fields private and final

```diff
-    private String fieldA;
-    private Date fieldB;
-    private Long fieldC;
-    private List<String> fieldD;
+    private final String fieldA;
+    private final Date fieldB;
+    private final Long fieldC;
+    private final List<String> fieldD;
```

#### :construction: The object has to be constructed in :one: single step.

```diff
-    public OldModel() {
+    public OldModel(
+            final String fieldA,
+            final Date fieldB,
+            final Long fieldC,
+            final List<String> fieldD) {
+        this.fieldA = fieldA;
+        this.fieldB = fieldB;
+        this.fieldC = fieldC;
+        this.fieldD = fieldD;
     }
```

#### :red_circle: Do not provide methods that can change the object state.

```diff
-    public void setFieldA(String fieldA) {
-        this.fieldA = fieldA;
-    }
.
. // Remove all the setters
.
```

#### :fast_forward: If any of the fields is a mutable object, provide a defensive copy of that object instead.

```diff
+   public Date getFieldB() {
+       return new Date(fieldB.getTime()); // Easy to forget about this :(
+   }
+
+   public List<String> getFieldD() {
+       return Collections.unmodifiableList(fieldD); // This is not great :(
+   }
```

### About manually :raised_hand: doing this :triumph:
Well, this is much work, and it is error-prone as well, and even when we can make the IDE vomit all that code for us, we still need to check and modify certain things.

#### :no_good: And this is the result:

```java
import java.util.Collections;
import java.util.Date;
import java.util.List;
import java.util.Objects;

public final class OldModel {

    private final String fieldA;
    private final Date fieldB;
    private final Long fieldC;
    private final List<String> fieldD;

    public OldModel(
            final String fieldA,
            final Date fieldB,
            final Long fieldC,
            final List<String> fieldD) {
        this.fieldA = fieldA;
        this.fieldB = fieldB;
        this.fieldC = fieldC;
        this.fieldD = fieldD;
    }

    public String getFieldA() {
        return this.fieldA;
    }

    public Date getFieldB() {
        return new Date(this.fieldB.getTime()); // Easy to forget about this :(
    }

    public Long getFieldC() {
        return this.fieldC;
    }

    public List<String> getFieldD() {
        return Collections.unmodifiableList(this.fieldD); // This is not great :(
    }

    // toSting, equals and hashCode omitted
}
```

### :trophy: Let's do it now using the easy way

Immutables Library

> Java annotation processors to generate simple, safe and consistent value objects. Do not repeat yourself, try Immutables, the most comprehensive tool in this field!

Include the [immutables.org](http://immutables.github.io/) dependency to your project:

#### Maven Dependency:

```xml
<dependency>
    <groupId>org.immutables</groupId>
    <artifactId>value</artifactId>
    <version>2.8.3</version>
    <scope>provided</scope>
</dependency>
```

#### :construction_worker: Create the immutable object

```java
import org.immutables.value.Value;

import java.util.Date;
import java.util.List;

@Value.Immutable
public interface NewModel {

    String fieldA();

    Date fieldB();

    Long fieldC();

    List<String> fieldD();
}
```

#### :nut_and_bolt: Compile and Enjoy

After compilation, the annotation processor would generate the following code for you:

> A generated `final` class that extends a manually-written interface value type and implements all declared accessor methods as well as supporting **fields, methods, constructors, and a builder class**.

> An immutable implementation class implements abstract attribute accessors for scalar primitive and object reference types, with special support provided for collection attributes and other types. java.lang.Object's `equals`, `hashCode`, and `toString` methods are overridden and fully dependent on attribute values rather than on object identity.

> Immutable implementation classes are the primary (but not the only) source code artifacts generated by the Immutables annotation processor.

#### :pencil2: How to use the generated immutable class

```java
public class App {
    public static void main(String[] args) {
        final ImmutableNewModel model = ImmutableNewModel.builder()
                .fieldA("A")
                .fieldB(new Date())
                .fieldC(1L)
                .addFieldD("a", "b", "d")
                .addFieldD("e")
                .build();
        System.out.println(model);
    }
}
```

Output:

```json
NewModel{fieldA=A, fieldB=Sat Apr 11 15:53:26 PDT 2020, fieldC=1, fieldD=[a, b, d, e]}
```

#### :triangular_ruler: A few comparisons:

|                            |   OldModel    |      NewModel      |
|:---------------------------|:-------------:|:------------------:|
| Lines of code to maintain  |      69       |         16         |
| Lines of code to generated |       0       |        377         |
| Defensive copy of fields   | :interrobang: | :white_check_mark: |
| Fluent API for copy        |      :x:      | :white_check_mark: |
| Builder                    |      :x:      | :white_check_mark: |
| Nullability Checks         |      :x:      | :white_check_mark: |


#### :tada: The generated code

By default, on maven projects, the compiler would generate and auto import the generated code to and from `target/generated_sources` folder, notice that most of the time, we ignore the `target/` folder in version control systems (VCS) like git and mercurial. using a `.gitinore` file in the project. This code should not have to be pushed to the centralized VCS.

:exclamation::exclamation: **WARNING: It's a lot of code**
<details><summary>See the code</summary>
<p>
```java
package com.groupon.api.talks;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;
import java.util.Date;
import java.util.List;
import java.util.Objects;
import org.immutables.value.Generated;

/**
 * Immutable implementation of {@link NewModel}.
 * <p>
 * Use the builder to create immutable instances:
 * {@code ImmutableNewModel.builder()}.
 */
@Generated(from = "NewModel", generator = "Immutables")
@SuppressWarnings({"all"})
@javax.annotation.Generated("org.immutables.processor.ProxyProcessor")
public final class ImmutableNewModel implements NewModel {
  private final String fieldA;
  private final Date fieldB;
  private final Long fieldC;
  private final List<String> fieldD;

  private ImmutableNewModel(
      String fieldA,
      Date fieldB,
      Long fieldC,
      List<String> fieldD) {
    this.fieldA = fieldA;
    this.fieldB = fieldB;
    this.fieldC = fieldC;
    this.fieldD = fieldD;
  }

  /**
   * @return The value of the {@code fieldA} attribute
   */
  @Override
  public String fieldA() {
    return fieldA;
  }

  /**
   * @return The value of the {@code fieldB} attribute
   */
  @Override
  public Date fieldB() {
    return fieldB;
  }

  /**
   * @return The value of the {@code fieldC} attribute
   */
  @Override
  public Long fieldC() {
    return fieldC;
  }

  /**
   * @return The value of the {@code fieldD} attribute
   */
  @Override
  public List<String> fieldD() {
    return fieldD;
  }

  /**
   * Copy the current immutable object by setting a value for the {@link NewModel#fieldA() fieldA} attribute.
   * An equals check used to prevent copying of the same value by returning {@code this}.
   * @param value A new value for fieldA
   * @return A modified copy of the {@code this} object
   */
  public final ImmutableNewModel withFieldA(String value) {
    String newValue = Objects.requireNonNull(value, "fieldA");
    if (this.fieldA.equals(newValue)) return this;
    return new ImmutableNewModel(newValue, this.fieldB, this.fieldC, this.fieldD);
  }

  /**
   * Copy the current immutable object by setting a value for the {@link NewModel#fieldB() fieldB} attribute.
   * A shallow reference equality check is used to prevent copying of the same value by returning {@code this}.
   * @param value A new value for fieldB
   * @return A modified copy of the {@code this} object
   */
  public final ImmutableNewModel withFieldB(Date value) {
    if (this.fieldB == value) return this;
    Date newValue = Objects.requireNonNull(value, "fieldB");
    return new ImmutableNewModel(this.fieldA, newValue, this.fieldC, this.fieldD);
  }

  /**
   * Copy the current immutable object by setting a value for the {@link NewModel#fieldC() fieldC} attribute.
   * An equals check used to prevent copying of the same value by returning {@code this}.
   * @param value A new value for fieldC
   * @return A modified copy of the {@code this} object
   */
  public final ImmutableNewModel withFieldC(Long value) {
    Long newValue = Objects.requireNonNull(value, "fieldC");
    if (this.fieldC.equals(newValue)) return this;
    return new ImmutableNewModel(this.fieldA, this.fieldB, newValue, this.fieldD);
  }

  /**
   * Copy the current immutable object with elements that replace the content of {@link NewModel#fieldD() fieldD}.
   * @param elements The elements to set
   * @return A modified copy of {@code this} object
   */
  public final ImmutableNewModel withFieldD(String... elements) {
    List<String> newValue = createUnmodifiableList(false, createSafeList(Arrays.asList(elements), true, false));
    return new ImmutableNewModel(this.fieldA, this.fieldB, this.fieldC, newValue);
  }

  /**
   * Copy the current immutable object with elements that replace the content of {@link NewModel#fieldD() fieldD}.
   * A shallow reference equality check is used to prevent copying of the same value by returning {@code this}.
   * @param elements An iterable of fieldD elements to set
   * @return A modified copy of {@code this} object
   */
  public final ImmutableNewModel withFieldD(Iterable<String> elements) {
    if (this.fieldD == elements) return this;
    List<String> newValue = createUnmodifiableList(false, createSafeList(elements, true, false));
    return new ImmutableNewModel(this.fieldA, this.fieldB, this.fieldC, newValue);
  }

  /**
   * This instance is equal to all instances of {@code ImmutableNewModel} that have equal attribute values.
   * @return {@code true} if {@code this} is equal to {@code another} instance
   */
  @Override
  public boolean equals(Object another) {
    if (this == another) return true;
    return another instanceof ImmutableNewModel
        && equalTo((ImmutableNewModel) another);
  }

  private boolean equalTo(ImmutableNewModel another) {
    return fieldA.equals(another.fieldA)
        && fieldB.equals(another.fieldB)
        && fieldC.equals(another.fieldC)
        && fieldD.equals(another.fieldD);
  }

  /**
   * Computes a hash code from attributes: {@code fieldA}, {@code fieldB}, {@code fieldC}, {@code fieldD}.
   * @return hashCode value
   */
  @Override
  public int hashCode() {
    int h = 5381;
    h += (h << 5) + fieldA.hashCode();
    h += (h << 5) + fieldB.hashCode();
    h += (h << 5) + fieldC.hashCode();
    h += (h << 5) + fieldD.hashCode();
    return h;
  }

  /**
   * Prints the immutable value {@code NewModel} with attribute values.
   * @return A string representation of the value
   */
  @Override
  public String toString() {
    return "NewModel{"
        + "fieldA=" + fieldA
        + ", fieldB=" + fieldB
        + ", fieldC=" + fieldC
        + ", fieldD=" + fieldD
        + "}";
  }

  /**
   * Creates an immutable copy of a {@link NewModel} value.
   * Uses accessors to get values to initialize the new immutable instance.
   * If an instance is already immutable, it is returned as is.
   * @param instance The instance to copy
   * @return A copied immutable NewModel instance
   */
  public static ImmutableNewModel copyOf(NewModel instance) {
    if (instance instanceof ImmutableNewModel) {
      return (ImmutableNewModel) instance;
    }
    return ImmutableNewModel.builder()
        .from(instance)
        .build();
  }

  /**
   * Creates a builder for {@link ImmutableNewModel ImmutableNewModel}.
   * <pre>
   * ImmutableNewModel.builder()
   *    .fieldA(String) // required {@link NewModel#fieldA() fieldA}
   *    .fieldB(Date) // required {@link NewModel#fieldB() fieldB}
   *    .fieldC(Long) // required {@link NewModel#fieldC() fieldC}
   *    .addFieldD|addAllFieldD(String) // {@link NewModel#fieldD() fieldD} elements
   *    .build();
   * </pre>
   * @return A new ImmutableNewModel builder
   */
  public static ImmutableNewModel.Builder builder() {
    return new ImmutableNewModel.Builder();
  }

  /**
   * Builds instances of type {@link ImmutableNewModel ImmutableNewModel}.
   * Initialize attributes and then invoke the {@link #build()} method to create an
   * immutable instance.
   * <p><em>{@code Builder} is not thread-safe and generally should not be stored in a field or collection,
   * but instead used immediately to create instances.</em>
   */
  @Generated(from = "NewModel", generator = "Immutables")
  public static final class Builder {
    private static final long INIT_BIT_FIELD_A = 0x1L;
    private static final long INIT_BIT_FIELD_B = 0x2L;
    private static final long INIT_BIT_FIELD_C = 0x4L;
    private long initBits = 0x7L;

    private String fieldA;
    private Date fieldB;
    private Long fieldC;
    private List<String> fieldD = new ArrayList<String>();

    private Builder() {
    }

    /**
     * Fill a builder with attribute values from the provided {@code NewModel} instance.
     * Regular attribute values will be replaced with those from the given instance.
     * Absent optional values will not replace present values.
     * Collection elements and entries will be added, not replaced.
     * @param instance The instance from which to copy values
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder from(NewModel instance) {
      Objects.requireNonNull(instance, "instance");
      fieldA(instance.fieldA());
      fieldB(instance.fieldB());
      fieldC(instance.fieldC());
      addAllFieldD(instance.fieldD());
      return this;
    }

    /**
     * Initializes the value for the {@link NewModel#fieldA() fieldA} attribute.
     * @param fieldA The value for fieldA
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder fieldA(String fieldA) {
      this.fieldA = Objects.requireNonNull(fieldA, "fieldA");
      initBits &= ~INIT_BIT_FIELD_A;
      return this;
    }

    /**
     * Initializes the value for the {@link NewModel#fieldB() fieldB} attribute.
     * @param fieldB The value for fieldB
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder fieldB(Date fieldB) {
      this.fieldB = Objects.requireNonNull(fieldB, "fieldB");
      initBits &= ~INIT_BIT_FIELD_B;
      return this;
    }

    /**
     * Initializes the value for the {@link NewModel#fieldC() fieldC} attribute.
     * @param fieldC The value for fieldC
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder fieldC(Long fieldC) {
      this.fieldC = Objects.requireNonNull(fieldC, "fieldC");
      initBits &= ~INIT_BIT_FIELD_C;
      return this;
    }

    /**
     * Adds one element to {@link NewModel#fieldD() fieldD} list.
     * @param element A fieldD element
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder addFieldD(String element) {
      this.fieldD.add(Objects.requireNonNull(element, "fieldD element"));
      return this;
    }

    /**
     * Adds elements to {@link NewModel#fieldD() fieldD} list.
     * @param elements An array of fieldD elements
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder addFieldD(String... elements) {
      for (String element : elements) {
        this.fieldD.add(Objects.requireNonNull(element, "fieldD element"));
      }
      return this;
    }


    /**
     * Sets or replaces all elements for {@link NewModel#fieldD() fieldD} list.
     * @param elements An iterable of fieldD elements
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder fieldD(Iterable<String> elements) {
      this.fieldD.clear();
      return addAllFieldD(elements);
    }

    /**
     * Adds elements to {@link NewModel#fieldD() fieldD} list.
     * @param elements An iterable of fieldD elements
     * @return {@code this} builder for use in a chained invocation
     */
    public final Builder addAllFieldD(Iterable<String> elements) {
      for (String element : elements) {
        this.fieldD.add(Objects.requireNonNull(element, "fieldD element"));
      }
      return this;
    }

    /**
     * Builds a new {@link ImmutableNewModel ImmutableNewModel}.
     * @return An immutable instance of NewModel
     * @throws java.lang.IllegalStateException if any required attributes are missing
     */
    public ImmutableNewModel build() {
      if (initBits != 0) {
        throw new IllegalStateException(formatRequiredAttributesMessage());
      }
      return new ImmutableNewModel(fieldA, fieldB, fieldC, createUnmodifiableList(true, fieldD));
    }

    private String formatRequiredAttributesMessage() {
      List<String> attributes = new ArrayList<>();
      if ((initBits & INIT_BIT_FIELD_A) != 0) attributes.add("fieldA");
      if ((initBits & INIT_BIT_FIELD_B) != 0) attributes.add("fieldB");
      if ((initBits & INIT_BIT_FIELD_C) != 0) attributes.add("fieldC");
      return "Cannot build NewModel, some of required attributes are not set " + attributes;
    }
  }

  private static <T> List<T> createSafeList(Iterable<? extends T> iterable, boolean checkNulls, boolean skipNulls) {
    ArrayList<T> list;
    if (iterable instanceof Collection<?>) {
      int size = ((Collection<?>) iterable).size();
      if (size == 0) return Collections.emptyList();
      list = new ArrayList<>();
    } else {
      list = new ArrayList<>();
    }
    for (T element : iterable) {
      if (skipNulls && element == null) continue;
      if (checkNulls) Objects.requireNonNull(element, "element");
      list.add(element);
    }
    return list;
  }

  private static <T> List<T> createUnmodifiableList(boolean clone, List<T> list) {
    switch(list.size()) {
    case 0: return Collections.emptyList();
    case 1: return Collections.singletonList(list.get(0));
    default:
      if (clone) {
        return Collections.unmodifiableList(new ArrayList<>(list));
      } else {
        if (list instanceof ArrayList<?>) {
          ((ArrayList<?>) list).trimToSize();
        }
        return Collections.unmodifiableList(list);
      }
    }
  }
}
```
</p>
</details>

## Conclusion
The Java language can be even more verbose if we don't use the proper tools for the job, and for years, code generation has been a useful solution to make our life easier in the Java ecosystem. Reaching a good level of immutability in our codebases requires much effort when doing it manually, and it's susceptible to inadvertent mistakes, to avoid that, and make our codebase also smaller (less code fewer bugs) we can should the [Immutables.org](http://immutables.github.io/) library.
