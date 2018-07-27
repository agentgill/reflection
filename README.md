# Apex Reflection Convention

Apex lacks native reflection, but it's still semi-possible with a shared convention. This is interesting for ISV and cross namespace applications. We can avoid extension packages and compile-time bindings on invoked code.

### Goals

- Be discoverable
- Avoid package dependencies

### Introducing StubProvider

Developers can achieve cheap reflection in Apex by using the StubProvider system interface as a shared entry point.

```c#
Object handleMethodCall(
    Object stub,
    String method,
    Type returnType,
    List<Type> argTypes,
    List<String> argNames,
    List<Object> argValues
)
```

The class to be executed is instrumented with `handleMethodCall` which delegates to methods made available.

### Discovering Methods

```c#
Type reflector = Type.forName('my_ns.Extension')
StubProvider impl = (StubProvider)reflector.newInstance();
List<List<Object>> supportedMethods = impl.handleMethodCall(impl, 'reflect', null, null, null, null);
```

### Invoking Methods

```c#
String result = impl.handleMethodCall(impl, 'exampleOne', null, null, null, null);
```

### Defining Methods

```c#
global class Extension implements StubProvider {
    
    /**
     * Actual method
     */
    String exampleOne(String arg1)
    {
      return arg1 + arg1;
    }

    /**
     * Actual method
     */
    Decimal exampleTwo(Decimal arg2)
    {
      return arg2 * arg2;
    }

    /**
     * Advertises supported methods
     */
    List<List<Object>> reflect()
    {
        return new List<List<Object>>
        {
            new List<Object>{
                'exampleOne',                 // method
                String.class,                 // return type
                new List<Type>{String.class}, // arg types
                new List<String>{'arg1'}      // arg names
            },
            new List<Object>{
                'exampleTwo',                  // method
                String.class,                  // return type
                new List<Type>{Decimal.class}, // arg types
                new List<String>{'arg2'}       // arg names
            }
        }
    }

    /**
     * Dispatches actual methods
     */
    global Object handleMethodCall(Object stub, String method, Type returnType, List<Type> argTypes, List<String> argNames, List<Object> argValues)
    {
       if (method == 'reflect') return this.reflect();
       if (method == 'exampleOne') return this.exampleOne((String)argValues[0]);
       if (method == 'exampleTwo') return this.exampleTwo((Decimal)argValues[0]);
       throw new StringException('method not implemented');
    }
    
}
```
