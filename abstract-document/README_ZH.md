---
layout: pattern
标题: 抽象文档
文件夹: abstract-document
permalink: /patterns/abstract-document/
分类: Structural
tags: 
 - Extensibility
---

## Intent

使用动态属性，在保持类型安全的同时，实现非类型语言的灵活性。

## 解释

抽象文档模式可以处理额外的非静态属性。该模式使用特质的概念来实现类型安全，并将不同类的属性分离为一组接口。

真实世界的例子

> 考虑一辆由多个部件组成的汽车。然而，我们不知道具体的汽车是否真的拥有所有的部件，或者只是其中的一部分。我们的汽车是动态的，非常灵活的。


简单来说

> 抽象文档模式允许在对象不知情的情况下附加属性。

维基百科说

> 一种面向对象的结构设计模式，用于在松散类型的键值存储中组织对象，并使用类型化的视图暴露数据。该模式的目的是在强类型语言中实现组件之间的高度灵活性，
> 在这种情况下，新的属性可以快速添加到对象树中，而不会失去类型安全的支持。该模式利用特质将一个类的不同属性分成不同的接口。

**程序实例**。

首先让我们定义基础类`Document`和`AbstractDocument`。它们基本上使对象持有一个属性图和任意数量的子对象。

```java
public interface Document {

  Void put(String key, Object value);

  Object get(String key);

  <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor);
}

public abstract class AbstractDocument implements Document {

  private final Map<String, Object> properties;

  protected AbstractDocument(Map<String, Object> properties) {
    Objects.requireNonNull(properties, "properties map is required");
    this.properties = properties;
  }

  @Override
  public Void put(String key, Object value) {
    properties.put(key, value);
    return null;
  }

  @Override
  public Object get(String key) {
    return properties.get(key);
  }

  @Override
  public <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor) {
    return Stream.ofNullable(get(key))
        .filter(Objects::nonNull)
        .map(el -> (List<Map<String, Object>>) el)
        .findAny()
        .stream()
        .flatMap(Collection::stream)
        .map(constructor);
  }
  ...
}
```
Next we define an enum `Property` and a set of interfaces for type, price, model and parts. This allows us to create
static looking interface to our `Car` class.

```java
public enum Property {

  PARTS, TYPE, PRICE, MODEL
}

public interface HasType extends Document {

  default Optional<String> getType() {
    return Optional.ofNullable((String) get(Property.TYPE.toString()));
  }
}

public interface HasPrice extends Document {

  default Optional<Number> getPrice() {
    return Optional.ofNullable((Number) get(Property.PRICE.toString()));
  }
}
public interface HasModel extends Document {

  default Optional<String> getModel() {
    return Optional.ofNullable((String) get(Property.MODEL.toString()));
  }
}

public interface HasParts extends Document {

  default Stream<Part> getParts() {
    return children(Property.PARTS.toString(), Part::new);
  }
}
```

现在我们准备介绍一下 `Car`。

```java
public class Car extends AbstractDocument implements HasModel, HasPrice, HasParts {

  public Car(Map<String, Object> properties) {
    super(properties);
  }
}
```

最后是我们如何在一个完整的例子中构造和使用`Car`。

```java
    LOGGER.info("Constructing parts and car");

    var wheelProperties = Map.of(
        Property.TYPE.toString(), "wheel",
        Property.MODEL.toString(), "15C",
        Property.PRICE.toString(), 100L);

    var doorProperties = Map.of(
        Property.TYPE.toString(), "door",
        Property.MODEL.toString(), "Lambo",
        Property.PRICE.toString(), 300L);

    var carProperties = Map.of(
        Property.MODEL.toString(), "300SL",
        Property.PRICE.toString(), 10000L,
        Property.PARTS.toString(), List.of(wheelProperties, doorProperties));

    var car = new Car(carProperties);

    LOGGER.info("Here is our car:");
    LOGGER.info("-> model: {}", car.getModel().orElseThrow());
    LOGGER.info("-> price: {}", car.getPrice().orElseThrow());
    LOGGER.info("-> parts: ");
    car.getParts().forEach(p -> LOGGER.info("\t{}/{}/{}",
        p.getType().orElse(null),
        p.getModel().orElse(null),
        p.getPrice().orElse(null))
    );

    // Constructing parts and car
    // Here is our car:
    // model: 300SL
    // price: 10000
    // parts: 
    // wheel/15C/100
    // door/Lambo/300
```

## 类图

![alt text](./etc/abstract-document.png "Abstract Document Traits and Domain")

## 适用性

在以下情况下使用抽象文档模式

* There is a need to add new properties on the fly
* You want a flexible way to organize domain in tree like structure
* 你要的是更松散的耦合系统

## Credits

* [维基百科：抽象文档模式](https://en.wikipedia.org/wiki/Abstract_Document_Pattern)
* [Martin Fowler: Dealing with properties](http://martinfowler.com/apsupp/properties.pdf)
* [Pattern-Oriented Software Architecture Volume 4: A Pattern Language for Distributed Computing (v. 4)](https://www.amazon.com/gp/product/0470059028/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=0470059028&linkId=e3aacaea7017258acf184f9f3283b492)
