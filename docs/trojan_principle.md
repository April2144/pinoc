# Principle of Trojan

This chapter covers the principle of the Trojan library.

When an Android app is built, the Trojan plugin modifies all of the Java methods.
After the modification, each method (the original method) becomes its variant.

At runtime, when a method is invoked, it is the variant of the original one that is actually invoked.
The variant passes the information about the invocation to Trojan,
which decides whether to replace or modify the original method.

To avoid the trouble caused by the Java classloader, Trojan does not adopt the Java classloader.
Thus the replacement or the modification of the original method is not written in Java.
Instead it is written in Zlang, a flexible programming language.
It is easy to convert the Java instructions into the Zlang instructions.

If Trojan decides to replace or modify the original method, it compiles the instructions of
the replacement or the modification by the Zlang compiler, after which the output of the compilation
is passed to the Zlang executor for execution.

### Code injection at compile time

When building an Android app, the Trojan plugin modifies all of the
Java methods by injecting a particular code snippet at the entrance
of each method to be modified.

Take the following as an example:

```
package com.iqiyi.trojantest;

public class Test {
    public String f(int a, int b) {
        if (a > b) {
            return "a";
        } else {
            return "b";
        }
    }
}
```

By code injection, the method becomes the following:

```
package com.iqiyi.trojantest;

public class Test {
    public String f(int a, int b) {
        Object var = Trojan.onEnterMethod("com/iqiyi/trojantest/Test", "f", "(II)Ljava/lang/String;", this, new Object[]{a, b});
        if(var != Library.NO_RETURN_VALUE) {
            if(var == null) {
                return null;
            }
            if(var instanceof String) {
                return (String)var;
            }
        }
        if (a > b) {
            return "a";
        } else {
            return "b";
        }
    }
}
```

The injected code snippet invokes a static method called `onEneterMethod` of the `Trojan` class.
`onEnterMethod` receives the information about the invocation, including the name of the class,
the name and the signature of the method being invoked, the object on which the method is being invoked,
and the parameters of the method being invoked.

Inside `onEnterMethod`, Trojan decides whether to execute some instructions and what to return after
the execution, according to a configuration, which specifies:

1. at the entrances of which methods, what instructions should be injected;

2. which methods should be replaced, and their replacements;

### Modification at runtime

At runtime, when Trojan is initialized, it reads a configuration which specified the modification of the app.

As mentioned above, when a method is being invoked, `Trojan.onEnterMethod` receives the corresponding
information about the invocation. Then Trojan does the following:

1. If the specified instructions should be injected at the entrance of the method,
Trojan executes the injected instructions and returns `Library.NO_RETURN_VALUE`.
Then the remaining instructions within the original method are executed.

2. If the method should be replaced, Trojan executes the instructions of its replacement and returns
the return value of the replacement to the original method. Then the original method returns it directly
and the remaining instructions are not executed. 

3. If the method should neither be modified nor replaced, `Trojan.onEnterMethod()` will simply
return `Library.NO_RETURN_VALUE`. Then the instructions of original method are executed.

Note that Trojan has a high performance in memory and CPU usage, so it will not
affect the memory and CPU usage of your app.

### Instructions written in Zlang

The traditional techniques always adopt a Java classloader to load a class and execute the corresponding
instructions within the class. However, the classloader causes the security problems and thus is not
allowed by some app stores.

To avoid the above problems, Trojan does not adopt the traditional classloader-based techniques, but
adopt a novel technique which is classloader-free. Specifically, Trojan executes the instructions
written not in Java, but in the Zlang programming language,
a flexible dynamically-typed programming language which runs on the JVM
and supports access to Java objects and interaction with Java at runtime.
It is easy to convert Java instructions into Zlang instructions.
```