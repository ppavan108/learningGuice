### Use @Nullable
To eliminate `NullPointerExceptions` in your codebase, you must be disciplined about null references. We've been successful at this by following and enforcing a simple rule:
  _Every parameter is non-null unless explicitly specified._
The [Guava: Google Core Libraries for Java](http://code.google.com/p/guava-libraries/) and [JSR-305](https://github.com/amaembo/jsr-305) have simple APIs to get a nulls under control. `Preconditions.checkNotNull` can be used to fast-fail if a null reference is found, and `@Nullable` can be used to annotate a parameter that permits the `null` value:
```java
import static com.google.common.base.Preconditions.checkNotNull;
import javax.annotation.Nullable;

public class Person {
  ...

  public Person(String firstName, String lastName, @Nullable Phone phone) {
    this.firstName = checkNotNull(firstName, "firstName");
    this.lastName = checkNotNull(lastName, "lastName");
    this.phone = phone;
  }
```
*Guice forbids null by default.* It will refuse to inject `null`, failing with a `ProvisionException` instead. If `null` is permissible by your class, you can annotate the field or parameter with `@Nullable`. Guice recognizes any `@Nullable` annotation, like [edu.umd.cs.findbugs.annotations.Nullable](http://findbugs.sourceforge.net/api/edu/umd/cs/findbugs/annotations/Nullable.html) or [javax.annotation.Nullable](https://github.com/amaembo/jsr-305/blob/master/ri/src/main/java/javax/annotation/Nullable.java).