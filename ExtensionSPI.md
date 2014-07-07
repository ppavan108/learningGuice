_Interaction between Guice SPI & Extensions_

### Extensions SPI

_New in Guice 3.0_

In Guice 2.0 we expanded our SPI to [[expose rich details about modules and injectors|InspectingModules]]. The elements API can inspect, extend and even rewrite Guice configuration. It's an essential tool used by GIN, Modules.override(), Grapher, Guiceberry's controllable injection, and others. In Guice 3.0 we expanded the SPI to include extensions!  Extensions such as Multibinder, Assisted Inject, Servlets and others can expose their details through programmatic inspection.

#### How Extensions Participate
An extension author needs to do two things.  First it must create a subinterface of `BindingTargetVisitor` that is specific to its extension.  For example, the servlet extension has `ServletModuleTargetVisitor`, and the Multibinder extension has `MultibindingsTargetVisitor`.  The extension must also bind the extension's provider instance to a `ProviderWithExtensionVisitor`.  The implementation of acceptExtensionVisitor must do an instanceof check on the `BindingTargetVisitor` and call the appropriate visit method.  If the visitor is an instance of the extension's visitor, it must visit the appropriate extension method, otherwise it must visit the normal `ProviderInstanceBinding`.
For example, this is what mapbinder does:
```java
  public <R, B> R acceptExtensionVisitor(BindingTargetVisitor<B, R> visitor,
      ProviderInstanceBinding<? extends B> binding) {
    if (visitor instanceof MultibindingsTargetVisitor) {
      return ((MultibindingsTargetVisitor<Map<K, V>, R>)visitor).visit(this);
    } else {
      return visitor.visit(binding);
    }
  }
```

#### How Users Participate
To visit an extension's custom methods, users must implement the extension's custom visitor interface and call `binding.acceptTargetVisitor(myVisitor)`.  For example:

```java
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(new PorscheModule());

    for (Binding<?> binding : injector.getBindings().values()) {
      System.out.println(binding.acceptTargetVisitor(new MultibindVisitor()));
    }
  }

  static class MultibindVisitor extends DefaultBindingTargetVisitor<?, String>
      implements MultibindingsTargetVisitor<?, String> {
    public String visit(MultibinderBinding<?> multibinder) {
      return "Key: " + multibinder.getSetKey()
          + " uses bindings: " + multibinder.getElements()
          + " permitsDuplicates: " + multibinder.permitsDuplicates();
    }
    public String visit(MapBinderBinding<?> mapbinder) {
      return "Key: " + mapbinder.getMapKey()
          + " uses entries: " + mapbinder.getEntries()
          + " permitsDuplicates: " + mapbinder.permitsDuplicates();
    }
    protected String visitOther(Binding<?> binding) {
      return binding.toString();
    }
  }
```