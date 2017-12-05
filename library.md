---
order: 3
icon: book
title: Library
subtitle: Build your own apps based on our implementation
---

<div class="alert alert-warning">
    <h4>Warning and Disclaimer</h4>
    <p>We recommend <strong>not</strong> to use the current Digital ID Library in production. See the <a href="#roadmap">roadmap</a> for more information.</p>
</div>

## Requirements

Since this is currently only a test version, you need [Apache Maven](https://maven.apache.org) to build the library from the source code. For compilation, [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html) or newer is required.

## Download

### Overview

The Digital ID Library is structured in such a way that clients and servers share as much code as possible:

![Project Dependencies]({{ site.baseurl }}/assets/graphics/dependencies/overview.svg)

### Versions

The current versions of these projects are:

* [Utility](https://github.com/synacts/digitalid-utility): {{ site.data.version.utility }}
* [Database](https://github.com/synacts/digitalid-database): {{ site.data.version.database }}
* [Core](https://github.com/synacts/digitalid-core): {{ site.data.version.core }}

### Cloning

Clone the [utility](https://github.com/synacts/digitalid-utility), [database](https://github.com/synacts/digitalid-database) and [core](https://github.com/synacts/digitalid-core) projects from [GitHub](https://github.com/):

```bash
git clone https://github.com/synacts/digitalid-utility.git
git clone https://github.com/synacts/digitalid-database.git
git clone https://github.com/synacts/digitalid-core.git
```

### Checkout

Checkout the development branch of each repository as follows:
```bash
git -C digitalid-utility checkout development
git -C digitalid-database checkout development
git -C digitalid-core checkout development
```

### Compilation

Build the source code as follows:

```bash
mvn -f digitalid-utility/pom.xml clean install
mvn -f digitalid-database/pom.xml clean install
mvn -f digitalid-core/pom.xml clean install
```

### Repository

If you only want to include the [utility](https://github.com/synacts/digitalid-utility) project for the benefits described in the [design](#design) section, you can add the following dependency to your [POM](https://maven.apache.org/pom.html) to load it from the [Maven Repository](https://mvnrepository.com/artifact/net.digitalid.utility/utility-all):

```xml
<dependency>
    <groupId>net.digitalid.utility</groupId>
    <artifactId>utility-all</artifactId>
    <version>0.7.0</version>
    <type>pom</type>
</dependency>
```

As soon as the other projects are also more mature, we are going to publish them in the [Maven Repository](http://mvnrepository.com) as well.

## Usage

### Dependencies

If you want to build a client, add the following dependencies to your [POM](https://maven.apache.org/pom.html) after having [compiled](#compilation) the sources:

```xml
<dependencies>
    
    <dependency>
        <groupId>net.digitalid.core</groupId>
        <artifactId>core-all</artifactId>
        <version>${project.version}</version>
    </dependency>
    
    <dependency>
        <groupId>net.digitalid.database</groupId>
        <artifactId>database-client</artifactId>
        <version>${project.version}</version>
    </dependency>
    
</dependencies>
```

### Initialization

Various configurations need to be initialized *before* the library can be used:

```java
import net.digitalid.utility.configuration.Configuration;

Configuration.initializeAllConfigurations();
```

### Client

Before we can create an identity, we first have to create the client with which we will manage the identity:

```java
import net.digitalid.core.client.Client;
import net.digitalid.core.client.ClientBuilder;
import net.digitalid.core.permissions.ReadOnlyAgentPermissions;

final @Nonnull Client client = ClientBuilder.withIdentifier("my.company.client").withDisplayName("Company App").withPreferredPermissions(ReadOnlyAgentPermissions.GENERAL_WRITE).build();
```

### Identifier

The following code checks whether a string is a valid identifier and whether such an identity already exists:

```java
import net.digitalid.core.identification.identifier.InternalNonHostIdentifier;

final @Nonnull String string = "user@digitalid.net";
if (InternalNonHostIdentifier.isValid(string)) {
    final @Nonnull InternalNonHostIdentifier identifier = InternalNonHostIdentifier.with(userString);
    if (!identifier.exists()) {
        // There exists no identity with this identifier and thus a new account can be opened
    }
}
```

### Identity

All actions and queries are performed on roles.
You can create a new identity with:

```java
import net.digitalid.core.account.OpenAccount;
import net.digitalid.core.client.role.NativeRole;
import net.digitalid.core.identification.identity.Category;

final @Nonnull NativeRole role = OpenAccount.of(Category.NATURAL_PERSON, identifier, client);
```

{% comment %}
Instead of creating a new identity, you can also request accreditation for an existing identity:

```java
final @Nonnull InternalNonHostIdentity identity = identifier.getIdentity();
final @Nonnull NativeRole role = client.accredit(identity, "secret");
```

Afterwards, you have to wait until this client is authorized by another client.
{% endcomment %}

### Attribute

Once you have such a role, it is easy to retrieve an attribute of that role:

```java
import net.digitalid.core.attribute.Attribute;
import net.digitalid.core.attribute.AttributeTypes;

final @Nonnull Attribute attribute = Attribute.of(role, AttributeTypes.NAME);
```

Until we provide better utility methods, setting the value of an attribute is a bit inconvenient:

```java
import net.digitalid.core.pack.Pack;
import net.digitalid.core.pack.PackConverter;
import net.digitalid.core.signature.Signature;
import net.digitalid.core.signature.SignatureBuilder;
import net.digitalid.core.signature.attribute.AttributeValue;
import net.digitalid.core.signature.attribute.UncertifiedAttributeValue;

final @Nonnull Signature<Pack> signature = SignatureBuilder.withObjectConverter(PackConverter.INSTANCE).withObject(Pack.pack(StringConverter.INSTANCE, "My Name", AttributeTypes.NAME)).withSubject(identifier).build();
final @Nonnull AttributeValue value = UncertifiedAttributeValue.with(signature);
attribute.value().set(value);
```

Setting the (in this example public) visibility of the attribute is then straightforward:

```java
import net.digitalid.core.expression.PassiveExpression;
import net.digitalid.core.expression.PassiveExpressionBuilder;

final @Nonnull PassiveExpression visibility = PassiveExpressionBuilder.withEntity(role).withString("everybody").build();
attribute.visibility().set(visibility);
```

### Caching

The attributes of other identities are cached locally and can be retrieved as follows:

```java
import net.digitalid.core.cache.CacheQueryBuilder;

final @Nonnull InternalNonHostIdentifier friend = new InternalNonHostIdentifier("friend@example.com");
final @Nonnull String name = CacheQueryBuilder.withConverter(StringConverter.INSTANCE).withRequestee(friend.resolve()).withRequester(role).withType(AttributeTypes.NAME).build().execute();
```

## Design

### Java Ecosystem

We used the programming language [Java](https://www.oracle.com/java/) for the Digital ID Library due to its wide adoption and the maturity of its ecosystem.

### Code Generation

The biggest downside of Java is that you have to write a lot of boilerplate code.
We managed to reduce this overhead to a minimum by heavily using [Java Annotation Processing](http://hannesdorfmann.com/annotation-processing/annotationprocessing101) to automatically generate builders, subclasses, converters and initializers.

For example,

```java
import net.digitalid.utility.annotations.method.Impure;
import net.digitalid.utility.annotations.method.Pure;
import net.digitalid.utility.generator.annotations.generators.GenerateBuilder;
import net.digitalid.utility.generator.annotations.generators.GenerateSubclass;
import net.digitalid.utility.rootclass.RootClass;

@GenerateBuilder
@GenerateSubclass
public abstract class MutableClass extends RootClass {
    
    @Pure
    public abstract int getValue();
    
    @Impure
    public abstract void setValue(int value);
    
}
```

generates the subclass (slightly edited here for conciseness)

```java
import java.util.Objects;

import javax.annotation.Nonnull;
import javax.annotation.Nullable;

import net.digitalid.utility.annotations.method.Pure;

class MutableClassSubclass extends MutableClass {
    
    private int value;
    
    @Override
    public int getValue() {
        return this.value;
    }
    
    @Override
    public void setValue(int value) {
        this.value = value;
    }
    
    MutableClassSubclass(int value) {
        this.value = value;
    }
    
    @Pure
    @Override
    public boolean equals(@Nullable Object object) {
        if (object == this) {
            return true;
        }
        if (object == null || !(object instanceof MutableClass)) {
            return false;
        }
        final @Nonnull MutableClass that = (MutableClass) object;
        boolean result = true;
        result = result && Objects.equals(this.getValue(), that.getValue());
        return result;
    }
    
    @Pure
    @Override
    public int hashCode() {
        int prime = 92_821;
        int result = 46_411;
        result = prime * result + Objects.hashCode(getValue());
        return result;
    }
    
    @Pure
    @Override
    public @Nonnull String toString() {
        return "MutableClass(value: " + getValue() + ")";
    }
    
}
```

and the builder (edited here for conciseness)

```java
import javax.annotation.Nonnull;

import net.digitalid.utility.annotations.method.Impure;
import net.digitalid.utility.annotations.method.Pure;

public class MutableClassBuilder {
    
    public static class InnerMutableClassBuilder {
        
        private InnerMutableClassBuilder() {}
        
        private int value = 0;
        
        @Impure
        public @Nonnull InnerMutableClassBuilder withValue(int value) {
            this.value = value;
            return this;
        }
        
        @Pure
        public @Nonnull MutableClass build() {
            return new MutableClassSubclass(value);
        }
        
    }
    
    @Pure
    public static @Nonnull InnerMutableClassBuilder withValue(int value) {
        return new InnerMutableClassBuilder().withValue(value);
    }
    
}
```

### Static Constructors

When we cannot or do not want to generate a builder automatically, we use static methods like `of(…)` and `with(…)` instead of public constructors.

This design principle has the following advantages:
- Possibility to return a cached object instead of a new one, which is important for the observer pattern.
- Possibility to return null instead of a new object, which is useful to propagate null values.
- Possibility to perform operations before having to call the constructor of the superclass.
- Possibility to choose a more descriptive name for the static constructor method.
- Possibility to construct objects of a subclass instead of the current class.
- Possibility to make use of these possibilities at some later point in time.
- And [NetBeans](https://netbeans.org) correctly checks non-null parameter assignments.

Besides slightly bigger classes, the biggest disadvantage seems to be that static methods are inherited and can thus lead to namespace pollution.

### Design by Contract

We follow the [design by contract](https://en.wikipedia.org/wiki/Design_by_contract) methodology.
When you annotate method parameters and return types, the subclass generator generates a subclass that overrides the method and performs the corresponding checks.

For example,

```java
import net.digitalid.utility.annotations.method.Impure;
import net.digitalid.utility.generator.annotations.generators.GenerateSubclass;
import net.digitalid.utility.validation.annotations.math.Positive;

@GenerateSubclass
public abstract class Test {
    
    @Impure
    public void setValue(@Positive int value) { /* … */}
    
}
```

generates the subclass

```java
import net.digitalid.utility.contracts.Require;
import net.digitalid.utility.validation.annotations.math.Positive;

class TestSubclass extends Test {
    
    @Override
    public void setValue(@Positive int value) {
        Require.that(value > 0).orThrow("The value has to be positive but was $.", value);
        super.setValue(value);
    }
    
}
```

### Type Safety

A built-in form of design by contract is the type safety provided by Java.
We tried to use class hierarchies wherever possible instead of relying on annotations to restrict a method parameter or a return type.
The downside is that it makes class hierarchies more complex.

{% comment %}
### Object Conversion

Both for SQL and [XDF]({% link protocol.html %}#_extensible_data_format)

### Functional Style

Functional iterables and failable interfaces

### Event Listening

[JavaFX-like](http://docs.oracle.com/javafx/2/binding/jfxpub-binding.htm) property paradigm

### Encapsulation

Read-only collections, many immutable types to prevent aliasing

A design pattern that is used so extensively throughout the code so that it is worth to be mentioned here is `Freezable`. Objects that implement this interface can be changed until you `freeze()` them. From that point onwards they are immutable and can be shared among objects and threads without running into consistency problems. As you would expect by now, there are annotations to indicate whether the object denoted by a variable or parameter is `@Frozen` or `@NonFrozen`. Independent of their freezing state, such objects can be exposed through a `ReadOnly` interface (that only includes the methods that are `@Pure`).

### Transactions

Whether the database is accessed by a method or its called methods is indicated by the annotations `@Committing` and `@NonCommitting`. As their names suggest, they also declare whether the current transaction is committed by the method or not, which might not be desirable if the whole transaction should be rolled back on failure. Moreover, a method that is declared to be `@Committing` guarantees that the transaction is in a committed (or rolled back) state when the method is left without throwing an exception. If the last method in the try-block is committing, then the explicit commit at its end can be omitted.

### Flexibility

#### Platforms

Cross-compilation for iOS, retro-lambda for Android

#### Databases

Various SQL dialects (MySQL on server, SQLite on client)

## Documentation

You find the [Javadoc](http://en.wikipedia.org/wiki/Javadoc) of all non-private classes and methods at [docs.digitalid.net/documentation/](https://docs.digitalid.net/).
{% endcomment %}

## Roadmap

### Production (v0.7)

- Auto-incrementing columns
- Contact management
- Credential issuance
- Agent authorization
- Better test coverage
- Security review

### Self-Hosting (v0.8)

- Action pusher
- Audit functionality
- Domain verification
- Certificate issuance
- Client synchronization

### Functionality (v0.9)

- Hierarchical contexts
- Offline functionality
- Identity relocation
- Restricted agents

### History

There has been an older, [Apache Ant](http://ant.apache.org)-based version of this library.
A lot of things have changed and improved since then.
Some artifacts still need to be rewritten, though (see [core](#core)).

## Support

Please file any issues you encounter directly in the corresponding GitHub project ([utility](https://github.com/synacts/digitalid-utility/issues), [database](https://github.com/synacts/digitalid-database/issues) or [core](https://github.com/synacts/digitalid-core/issues)).
If you are uncertain to which project an issue belongs, use the [last one](https://github.com/synacts/digitalid-core/issues) or [contact us](mailto:support@digitalid.net) directly.
When you do so, please attach the current [log file](#troubleshooting).
(Since the log file might contain sensitive information, it might be a good idea to first have a look at it yourself.)

## License

The code of the Digital ID Library is available under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0), which is a [permissive software license](https://en.wikipedia.org/wiki/Permissive_software_licence) that lets you do anything you want with the code as long as you provide proper attribution and don’t hold us liable.

## Structure

The following graphics depict how the artifacts of each project depend on each other.
If you only need certain functionality, you can depend on the corresponding artifact directly.

### Utility

![Utility Dependencies]({{ site.baseurl }}/assets/graphics/dependencies/utility_v6.svg)

### Database

![Database Dependencies]({{ site.baseurl }}/assets/graphics/dependencies/database_v5.svg)

### Core

![Core Dependencies]({{ site.baseurl }}/assets/graphics/dependencies/core_v11.svg)

Please note that the artifacts with the red border largely still have to be written.
