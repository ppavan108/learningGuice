_Building Guice without AOP support_
### Optional AOP
Guice 1.0 was available in a single version that included AOP.

In Guice 2.0 and later, AOP is optional. If your platform doesn't support bytecode generation, you can download a version of Guice that doesn't include AOP support. This is most useful for mobile platforms like [Android](http://code.google.com/android/). This version also lacks **fast reflection** and **line numbers in error messages**. For this reason, we recommend Guice+AOP even in applications that don't use method interceptors.

#### Feature Comparison

|   | *Guice 1.0* | *Guice 2.0 no AOP* | *Guice 2.0* | *[GIN](http://code.google.com/p/google-gin)* |
|---|-------------|--------------------|-------------|----------------------------------------------|
| Library size | 544KB | 430KB | 652KB | 0KB (code gen) |
| High performance | ✔ | ✔ | ✔ | ✔ |
| Binding EDSL | ✔ | ✔ | ✔ | ✔ |
| Scopes | ✔ | ✔ | ✔ | ✔ |
| Typesafe | ✔ | ✔ | ✔ | ✔ |
| Fast reflection | ✔ ([cglib](http://cglib.sourceforge.net/)) | | ✔ ([cglib](http://cglib.sourceforge.net/)) | ✔ (code gen) |
| Line numbers in error messages  | ✔ |  | ✔ |  |
| Method interceptors  | ✔ | | ✔ ||  |
| Provider Methods |  | ✔ | ✔ | ✔ |
| Binding overrides |  | ✔ | ✔ |  |
| Tool-friendly SPI |  | ✔ | ✔ |  |
| Child Injectors |  | ✔ | ✔ |  |
| Servlet Support | Scopes Only | ✔ | ✔ |  |
| Error Reporting | Good | Better | Best | Best |

#### Building Guice without AOP
Guice has an ant task that creates a modified copy of the Guice source-tree. The copy uses the [munge](http://publicobject.com/2009/02/preprocessing-java-with-munge.html) preprocessor to remove all bytecode-dependent APIs. Use the following commands to build Guice without AOP support:
```bash
ant no_aop
cd build/no_aop/
ant dist 
```

Building with Maven automatically generates a no_aop artifact.