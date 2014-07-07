_Overview of Multibinder and MapBinder extensions_
### Multibindings
[Multibinder](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/multibindings/Multibinder.htm) and [MapBinder](http://google-guice.googlecode.com/svn/trunk/latest-javadoc/com/google/inject/multibindings/MapBinder.html) are intended for plugin-type architectures, where you've got several modules contributing Servlets, Actions, Filters, Components or even just names. 

#### Using Multibindings to host Plugins
Multibindings make it easy to support plugins in your application. Made popular by [IDEs](http://www.eclipseplugincentral.com) and [browsers](https://addons.mozilla.org/en-US/firefox/), this pattern exposes APIs for extending the behaviour of an application.

Neither the plugin consumer nor the plugin author need write much setup code for extensible applications with Guice. Simply define an interface, bind implementations, and inject sets of implementations! Any module can create a new Multibinder to contribute bindings to a set of implementations.  To illustrate, we'll use plugins to summarize ugly URIs like `http://bit.ly/1mzgW1` into something readable on Twitter.

First, we define an interface that plugin authors can implement. This is usually an interface that lends itself to several implementations. For this example, we would write a different implementation for each website that we could summarize.
```java
interface UriSummarizer {
  /** 
   * Returns a short summary of the URI, or null if this summarizer doesn't
   * know how to summarize the URI.
   */
  String summarize(URI uri);
}
```

Next, we'll get our plugin authors to implement the interface. Here's an implementation that shortens Flickr photo URLs:
```java
class FlickrPhotoSummarizer implements UriSummarizer {
  private static final Pattern PHOTO_PATTERN
      = Pattern.compile("http://www\.flickr\.com/photos/[^/]+/(\d+)/");

  public String summarize(URI uri) {
    Matcher matcher = PHOTO_PATTERN.matcher(uri.toString());
    if (!matcher.matches()) {
      return null;
    } else {
      String id = matcher.group(1);
      Photo photo = lookupPhoto(id);
      return photo.getTitle();
    }
  }
}
```

The plugin author registers their implementation using a multibinder. Some plugins may bind multiple implementations, or implementations of several extension-point interfaces.
```java
public class FlickrPluginModule extends AbstractModule {
  public void configure() {
    Multibinder<UriSummarizer> uriBinder = Multibinder.newSetBinder(binder(), UriSummarizer.class);
    uriBinder.addBinding().to(FlickrSummarizer.class);

    ... // bind plugin dependencies, such as our Flickr API key
  }
}
```

Now we can consume the services exposed by our plugins. In this case, we're summarizing tweets:
```java
public class TweetPrettifier {

  private final Set<UriSummarizer> summarizers;
  private final EmoticonImagifier emoticonImagifier;

  @Inject TweetPrettifier(Set<UriSummarizer> summarizers,
      EmoticonImagifier emoticonImagifier) {
    this.summarizers = summarizers;
    this.emoticonImagifier = emoticonImagifier;
  }

  public Html prettifyTweet(String tweetMessage) {
    ... // split out the URIs and call prettifyUri() for each
  }

  public String prettifyUri(URI uri) {
    // loop through the implementations, looking for one that supports this URI
    for (UrlSummarizer summarizer : summarizers) {
      String summary = summarizer.summarize(uri);
      if (summary != null) {
        return summary;
      }
    }

    // no summarizer found, just return the URI itself
    return uri.toString();
  }
}
```

_**Note:** The method `Multibinder.newSetBinder(binder, type)` can be confusing.  This operation creates a new binder, but doesn't override any existing bindings.  A binder created this way contributes to the existing Set of implementations for that type.  It would create a new set only if one is not already bound._


Finally we must register the plugins themselves. The simplest mechanism to do so is to list them programatically:
```java
public class PrettyTweets {
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(
        new GoogleMapsPluginModule(),
        new BitlyPluginModule(),
        new FlickrPluginModule()
        ...
    );

    injector.getInstance(Frontend.class).start();
  }
}
```
If it is infeasible to recompile each time the plugin set changes, the list of plugin modules can be loaded from a configuration file.

Note that this mechanism cannot load or unload plugins while the system is running. If you need to hot-swap application components, investigate [Guice's OSGi bindings](http://code.google.com/p/google-guice/wiki/OSGi).

#### Limitations
When you use PrivateModules with multibindings, all of the elements must be bound in the same environment. You cannot create collections whose elements span private modules. Otherwise injector creation will fail.

#### Inspecting Multibindings or MapBindings _(new in Guice 3.0)_
Sometimes you need to inspect the elements that make up a Multibinder or MapBinder.  For example, you may need a test that strips all elements of a MapBinder out of a series of modules.  You can visit a binding with a [MultibindingTargetVisitor](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/multibindings/MultibindingsTargetVisitor.html) to get details about Multibindings or MapBindings.  After you have an instance of a [MapBinderBinding](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/multibindings/MapBinderBinding.html) or a [MultibinderBinding](http://google-guice.googlecode.com/svn/trunk/javadoc/com/google/inject/multibindings/MultibinderBinding.html) you can learn more.

```java
   // Find the MapBinderBinding and use it to remove elements within it.
   Module stripMapBindings(Key<?> mapKey, Module... modules) {
     MapBinderBinding<?> mapBinder = findMapBinder(mapKey, modules);
     List<Element> allElements = Lists.newArrayList(Elements.getElements(modules));
     if (mapBinder != null) {
       List<Element> mapElements = getMapElements(mapBinder, modules);
       allElements.removeAll(mapElements);
     }
     return Elements.getModule(allElements);
  }

  // Look through all Elements in the module and, if the key matches,
  // then use our custom MultibindingsTargetVisitor to get the MapBinderBinding
  // for the matching binding.
  MapBinderBinding<?> findMapBinder(Key<?> mapKey, Module... modules) {
    for(Element element : Elements.getElements(modules)) {
      MapBinderBinding<?> binding =
          element.acceptVisitor(new DefaultElementVisitor<MapBinderBinding<?>>() {
            MapBinderBinding<?> visit(Binding<?> binding) {
              if(binding.getKey().equals(mapKey)) {
                return binding.acceptTargetVisitor(new Visitor());
              }
              return null;
            }
          });
      if (binding != null) {
        return binding;
      }
    }
    return null;
  }

  // Get all elements in the module that are within the MapBinderBinding.
  List<Element> getMapElements(MapBinderBinding<?> binding, Module... modules) {
    List<Element> elements = Lists.newArrayList();
    for(Element element : Elements.getElements(modules)) {
      if(binding.containsElement(elements)) {
        elements.add(element);
      }
    }
    return elements;
  }

  // A visitor that just returns the MapBinderBinding for the binding.
  class Visitor
      extends DefaultBindingTargetVisitor<Object, MapBinderBinding<?>>
      implements MultibindingsTargetVisitor<Object, MapBinderBinding<?>> {
    MapBinderBinding<?> visit(MapBinderBinding<?> mapBinder) {
      return mapBinder;
    }

    MapBinderBinding<?> visit(MultibinderBinding<?> multibinder) {
      return null;
    }
  }
```