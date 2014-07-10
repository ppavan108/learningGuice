_Inspecting what makes up a Module & examining Bindings_


### Elements SPI

Guice provides a rich service provider interface (SPI) for all the elements that make up a module.  You can use this SPI to learn what bindings are in a module and even rewrite some of those bindings.  [Modules.override](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/util/Modules.html#override(com.google.inject.Module...)) is written entirely using this SPI.

Examining Elements

The [Elements](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/Elements.html) class provides access to each [Element](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/Element.html) within a Module.  You can use this class to get individual elements from a module and take action upon them.  Each Element has an acceptVisitor method that takes an [ElementVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/ElementVisitor.html).  You can use [DefaultElementVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/DefaultElementVisitor.html) to simplify writing a visitor.

```java
  void warnAboutStaticInjections(Module... modules) {
    for(Element element : Elements.getElements(modules)) {
      element.acceptVisitor(new DefaultElementVisitor<Void>() {
          @Override public void visit(StaticInjectionRequest request) {
            System.out.println("You shouldn't be using static injection at: " + request.getSource());
          }
      });
    }
  }
```

#### Manipulating Modules

You can use the SPI to do anything from stripping out elements to rewriting elements to adding new elements.  When you've finished wiring your elements, you can turn them back into a module.

```java
  Module stripStaticInjections(Module... modules) {
    final List<Element> noStatics = Lists.newArrayList();
    for(Element element : Elements.getElements(modules)) {
      element.acceptVisitor(new DefaultElementVisitor<Void>() {
        @Override public void visit(StaticInjectionRequest request) {
          // override to not call visitOther
        }

        @Override public void visitOther(Element element) {
          noStatics.add(element);
        }
      });
    }
    return Elements.getModule(noStatics);
  }
```

#### Binding Binoculars

The most common Element is a [[Binding|Bindings]].  Bindings have more configuration than other elements and can be inspected in more ways.  You can visit a binding's scope using [Binding.acceptScopingVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Binding.html#acceptScopingVisitor)), or figure out what kind of binding it is using [Binding.acceptTargetVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Binding.html#acceptTargetVisitor).  Each of these methods have their own default visitors ([DefaultBindingScopingVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/DefaultBindingScopingVisitor.html) and [DefaultBindingTargetVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/DefaultBindingTargetVisitor.html), respectively) to make visiting easier.

Bindings can either be "Module bindings" or "Injector bindings".  Injector bindings are bindings retrieved from an injector (see the [Injector javadoc](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Injector.html)).  Module bindings are bindings retrieved using the Elements SPI.  The [Binding javadoc](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Binding.html) explains the difference between these in more detail.

#### Extensions

Guice 3.0 adds the ability for extensions to write their own SPI.  See [[ExtensionSPI]] for details.