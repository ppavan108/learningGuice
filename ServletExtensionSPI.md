_Using the servlet module SPI_

### ServletModule Extension SPI

_New in Guice 3.0_

Sometimes you want to examine your Modules and/or Bindings to inspect which URLs are serving which servlets or filters.  This is useful for tests or to assert preconditions in your code.  The !ServletModule extension SPI, built on the [[core extension SPI|ExtensionSPI]], lets you do this the same way you would inspect a binding using the normal [[elements SPI|InspectingModules]].

#### Inspecting Servlet Bindings

[ServletModuleTargetVisitor](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/servlet/ServletModuleTargetVisitor.html) is an extension to the core [BindingTargetVisitor](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/spi/BindingTargetVisitor.html).  You can implement this interface and call `binding.acceptTargetVisitor(myVisitor)` to learn details about servlet bindings.

```java
  boolean isBindingForUri(Binding<?> binding, String uri) {
    return binding.acceptTargetVisitor(new Visitor(uri));
  }

  private static class Visitor
      extends DefaultBindingTargetVisitor<Object, Boolean>
      implements ServletModuleTargetVisitor<Object, Boolean> {
    private final String uri;
    
    Visitor(String uri) {
      this.uri = uri;
    }

    @Override boolean visitOther(Binding<?> binding) {
      return false;
    }

    @Override boolean visit(InstanceFilterBinding binding) {
      return matchesUri(binding);
    } 

    @Override boolean visit(InstanceServletBinding binding) {
      return matchesUri(binding);
    }

    @Override boolean visit(LinkedFilterBinding binding) {
      return matchesUri(binding);
    }

    @Override boolean visit(LinkedServletBinding binding) {
      return matchesUri(binding);
    }

    private boolean matchesUri(ServletModuleBinding binding) {
      return binding.matchesUri(uri);
    }
  }
```

These visitors will work both on bindings retrieved from an Injector and bindings retrieved from Elements.