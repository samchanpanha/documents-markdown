# Understanding of Domain-Driven Design in DDD

**Original** **Jimoer** [Jimoer](javascript:void(0);)

 *August 27, 2025, 12:35*

## preface

**Domain Driven Design (DDD, Domain-Driven Design) is a set of complex software interface design methodology which is centered on business domain knowledge and driven by unified language and model. Its core idea is to deeply integrate technology implementations with business knowledge, and to make code a faithful reflection of business concepts, thereby delivering software that truly solves business problems on a continuous basis.**

## Concepts related to domain-driven design

### Domain model

**A domain model is an expression of a business concept in a program。The domain model can be used to design and understand the overall software structure. The concept of class in object-oriented design is an expression of domain model.Similarly, the UML modeling approach can be applied to the representation of domain models. **But it 's important to distinguish that a domain model is not a data model .

**Components of the domain model: entities, value objects, aggregations, domain services, domain events**

### (Entity)

`<span leaf="" class="js_darkmode__text__23">实体通常指具有唯一标识的具体对象或事物</span>`.. entities usually have their own life cycle and can be created, modified, and deleted.

**In a database, entities usually correspond to a single record in a database table, and each entity has a unique identifier (usually a primary key). In the code model, an entity is represented as an entity class, which contains the attributes and methods of the entity through which the entity's own business logic is implemented.**

### ValueObject

`<span leaf="" class="js_darkmode__text__32">值对象（Value Object）是通过对象属性值来识别的对象，它将多个相关属性组合为一个概念整体</span>`。

**Entity can be identified by ID, but value objects are identified by attributes. Value objects are usually immutable and cannot be modified once created, and can only be replaced by creating a new value object.**

**In a database, value objects usually correspond to a set of fields in a database table. Each value object does not have a unique identifier but describes its properties through a set set of Fields.**

**Value objects have two forms in code.**

1. **If the value object is a single property, it is defined directly as an attribute of the entity class;**
2. **If the value object is a set of attributes, it is designed as a Class class, Class will have a whole concept of multiple attributes to the property set, such a value object has no ID, will be referred to by the entity as a whole.**

### polymerization

**An aggregate is a collection of one or more strongly associated entities and value objects.**

**Entities and value objects in the domain model are treated as individuals, and the organization that enables entities and value objects to work together is the aggregation that ensures that the domain objects are consistent with the common business logic.**

**An aggregation has an aggregation root and a context boundary that defines which entities and value objects should be contained within an aggregation based on a single operational responsibility and the high cohesion principle, while the boundaries between an aggregation are loosely coupled. Microservices designed in this way are naturally "high cohesion, low coupling"**

#### Aggregate root

**The aggregate root is the root entity of the aggregate, which is not only the entity but also the manager of the aggregate, managing the lifecycle and integrity of other objects within the aggregate.**

**The polymer root is the only identifier in the polymer, the only entry point of the entire polymer, and all operations are carried out through the polymer root.**

### Area services

**Domain services are primarily used to handle business logic that does not fit in entities or value objects.**

**Area services have the following characteristics:**

* **The encapsulation of domain logic ** **: The domain service encapsulates the domain-specific business logic , which typically involves the interaction of multiple domain objects , and this encapsulation helps to keep the responsibilities of the entity and value objects single and clear**
* **Stateless ** **: Domain services are usually stateless, they do not store any business data, but operate domain objects to complete business logic。This helps keep services reusable and testable.**
* **Independence ** **: Domain services are typically independent of specific entities or value objects and provide a way to implement business rules that are independent of other parts of the domain model**
* **Reusability ** **: Domain services can be reused by different application requests , such as different application services orchestration or domain event processors**
* **Clear interface ** **: The interface of a domain service should clearly reflect the business capability it provides , and the return value of the parameter should be a domain object or a basic data type**

### Area events

**Domain events represent important events that occur in a domain and can be used to inform other domain objects or to decouple and collaborate across bounded contexts.**

**These are typically triggered by a change of state of the domain entities or aggregation roots. Domain events are not just changes in data, they also carry the context of the business and the intent of the business.**

**Area events have the following characteristics:**

* **Meaningful ** **: Domain events usually have a clear business meaning , such as " the user has placed an order " or " the product has been paid for . "**
* **Immutability ** **: Once a domain event is created, its state should not be changed。This helps to ensure consistency and reliability of the event.**
* **Time correlation ** **: Domain events often include a timestamp of when the event occurred , which helps to trace the sequence and timeline of events**
* **Relevance ** **: Domain events may be associated with specific domain entities and aggregate roots to help complete the event context**
* **Observability ** **: Domain events can be listened to and responded to by other parts of the system , helping to decouple the system**

**Unlike the MQ events we often hear about, domain events are not generally passed between distributed systems, but within individual microservices. Its greatest benefit, like MQ, is decoupling, uncoupling between domains by event, and a loosely coupled communication by publishing events without relying on specific implementation details.**

## Hierarchical architecture of domain model

**The hierarchical architecture of DDD is a four-tier architecture, from top to bottom: user interface layer, application layer, domain layer and foundation layer.**

**The layered architecture can be simply divided into two types, namely, strict layered architecture and loose layered architecture. In a strict layered architecture, a layer can only be coupled with a layer directly below it, whereas in a loose layered architecture a layer is allowed to be coupled to any layer below it.** ![Domain Driven Model Architecture Tiering](https://mmbiz.qpic.cn/mmbiz_png/51zXejl2tKxXRvCYibTAf9Vficb0rW1DSnK5CVVIFcicz80qkhu0RWzrySqib6eeY7KLTqFQgqgZaCtj5nHV0SWblQ/640?wx_fmt=png&from=appmsg)

**The advantages of this layered structure are:**

1. **Developers can focus on only one layer of the entire structure.**
2. **A new implementation can be easily replaced with an existing implementation.**
3. **You can reduce the dependencies between layers.**
4. **It is conducive to standardization.**
5. **It facilitates the reuse of layers of logic.**

**There are, of course, some drawbacks:**

* **It reduces the performance of the system. This is obvious because the middle layer has been added, but it can be improved by caching mechanisms.**
* **This may lead to a modification of the cascade. This modification is particularly reflected in the top-down direction, although it can be improved by relying on the inversion.**

### The layered architecture of the onion

**An onion architecture is a layer-by-layer architecture like an onion, from the outside to the inside, as shown below**![The layered architecture of the onion](https://mmbiz.qpic.cn/mmbiz_png/51zXejl2tKxXRvCYibTAf9Vficb0rW1DSngbomZ8f32RVPjUtATL3mLN9ITwQPk86BQgJUNYe7YqGdIZ411cOlCA/640?wx_fmt=png&from=appmsg)

### Six-sided layered architecture

**The hexagonal architecture, proposed by Alistair Cockburn in 2005, solved the problem posed by traditional layered architecture, which is actually a layered architecture too, but not up or down, but becomes internal and external. The hexagonal architecture is also known as the port-adaptor architecture:**

![Six-sided layered architecture](https://mmbiz.qpic.cn/mmbiz_png/51zXejl2tKxXRvCYibTAf9Vficb0rW1DSnnhsvzhYricljSrFzjREibDXiaXDf3RgPtibbia7AAW0EPV0ibOcWhAL04NwQ/640?wx_fmt=png&from=appmsg)
Six-sided layered architecture

**Although the DDD Layered Architecture, Onion Architecture, and Hexagon Architecture have different representations, the design philosophy of all three models is a perfect embodiment of the principle of high cohesion and low coupling in microservices architecture, and all are domain model centric.**

## Domain model design related

### Storage

**A repository is an interface for managing the creation, updating, and persistence of domain objects.**

**The repository acts as an intermediary between the domain model and the data store. It hides the details of the underlying data access, provides a consistent interface and abstraction, and makes the access and persistence of domain objects simple and uniform.**

### factory

**A factory is a class or method whose responsibility is to build a domain model (entity or value object). Factories can build different domain models with different business parameters. Simple business logic implementations can be implemented without the use of a factory. The implementation of a factory does not have to be in the form of a class, but can also be a method that has the function of a plant.**

### Models of anaemia

**Anemic Domain Model is a model that separates data from behavior, where the data is held by objects and the behavior is provided by external services. Its main feature is the lack of behavior of domain objects, usually containing only data attributes and simple getter and setter methods, while business logic is placed in the service layer. For example, the following products**

```
/**
 * 商品类
 */
publicclass Goods {

    private String name;
    private BigDecimal price;
    privateint count;

    public Goods(String name, BigDecimal price, int count) {
        this.name = name;
        this.price = price;
        this.count = count;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }
}
```

**Product behaviour**

```
public class GoodsService {

    public void updatePrice(Goods goods, BigDecimal price){
        if(Objects.isNull(price) || BigDecimal.ZERO.compareTo(price) > 0){
            throw new RuntimeException("价格不能小于0");
        }
        goods.setPrice(price);
    }

}
```

### Embryonic Model

**In domain-driven design, the congested model is a design pattern that emphasizes encapsulating data and business logic within domain objects.**

**In contrast to anaemia models, congested models combine data with behavior, allowing domain objects to be more than just containers of data, but also to contain business logic related to their state.**

```
/**
 * 商品类
 */
publicclass Goods {

    private String name;
    private BigDecimal price;
    privateint count;

    public Goods(String name, BigDecimal price, int count) {
        this.name = name;
        this.price = price;
        this.count = count;
    }

    public void updatePrice(Goods goods, BigDecimal price){
        if(Objects.isNull(price) || BigDecimal.ZERO.compareTo(price) > 0){
            thrownew RuntimeException("价格不能小于0");
        }
        goods.setPrice(price);
    }
}
```

### Pros and Cons and Use Cases

#### The advantages of anaemia models

1. **Separation of data from behavior reduces the complexity of the object.**
2. **It can improve code reuse and testability.**
3. **Existing services and frameworks could be made better use of.**

#### Disadvantages of anaemia models

1. **Objects lack encapsulation and are prone to coupling and vulnerability.**
2. **Business logic is scattered across multiple classes, making it difficult to maintain and understand.**
3. **Over-reliance on external services can lead to system instability.**

#### The advantages of the congestive model

1. **Object-oriented design with good encapsulation and maintainability.**
2. **Domain objects contain business logic and are easy to understand and extend.**
3. **Over-reliance on external services can be avoided and system stability improved.**

#### Disadvantages of the congestive model

1. **Requires an understanding of the model for better development and is costly to get started**
2. **Collaboration between objects may increase, leading to complexity in the design.**
3. **The state of the object may become inconsistent and requires special attention.**

**In general, anemia models can be used for smaller applications or simple business processes; For larger application systems or complex business processes, a congested model is recommended.**

### Preservative layer

**Anti-corrosion layer, which is used to protect the internal models and business logic of the system from external systems or services. Its main purpose is to provide a clear interface for isolating changes in external systems to prevent them from corrupting or negatively affecting the internal domain model.**

**For example, when a business module needs to rely on the data or function provided by a third-party system, our common strategy is to directly use the external system API, data structure. The problem is that the use of an external system is affected by the quality problems of the external system, thereby "deteriorating" the problems of the design itself.**

![](https://mmbiz.qpic.cn/mmbiz_png/51zXejl2tKxXRvCYibTAf9Vficb0rW1DSnaZUm7OFNMt9mbbJ9tnYwZWgUXb2aKMqo1Vr6CahR0VR2DpB8jiaqRgw/640?wx_fmt=png&from=appmsg)

**So the solution is to add a middle layer between the two systems to isolate the dependency of the third-party system, to communicate conversion and semantic isolation of the Third-Party System, and this middle layer we call a preservative layer.**

![](https://mmbiz.qpic.cn/mmbiz_png/51zXejl2tKxXRvCYibTAf9Vficb0rW1DSnBBoFJS3xFicibdSgpcKIqm2sLqoo8mqnU6Fw9fnOKeonojGiaQ2KGfgZA/640?wx_fmt=png&from=appmsg)

**A middle layer is added between the two systems, which is similar to the adapter mode to solve the ducking of interface differences, and the interface conversion is one-way (i.e., the interface transition is carried out from the call direction of the being called party); The corrosion layer emphasizes the semantic decoupling of the two subsystems, and the interface transformation is two-way.**

**The role of the embalming layer:**

* **Decouple the systems of both parties, isolate the effects of changes on both parties and allow both parties to evolve independently.**
* **The anti-corrosion layer allows other external systems to achieve seamless integration with the system without changing the domain layer of the existing system, thus reducing the workload of system integration.**
