# mnemo
Mnemo (short for Μνημοσύνη) is an extendible and easy to use bridge among programming languages. Mnemo allows to define modules, implemented
in any supported programming language, that can be used in any supported programming language.

## Modules and types
A module is defined through two files: 
- A ".mnif" file that defines the interface of the module and all the wrapped types
- A ".mnim" file that defines what should happen when the interface is accessed.
 
The mnif file would look like this:
```
guest Ruby;

namespace com.example {
  module Example {
    service test1(a:int@generics, b:string@generics):string@generics;
    service test2():void;
  }
  
  type ExObj as wrapper of ExampleObject;
  type ExObj2 as wrapper of ExampleObject2 {
    method test():string@generics;
  }
}
```
- `guest` defines the language used in the guest environment;
- `namespace` defines the namespace of all the enclosing objects;
- `module` defines a new module;
- `service` defines a service of the module. Types are mandatory and they have to be mnemo types;
- `type` defines a new mnemo type;
- `method` defines a method of a wrapped object, that allows to expose methods of the wrapped type.

The mnim file looks like this:
```
namespace com.example {
  define Example.test1(a, b) {
    ...
  }
  
  define Example.test2() {
    ...
  }
  
  define ExObj2.test() {
    ...
  }
}
```

There is no need to repeat the types of parameters/services/methods. Note that the mnim file contains just `define` statements,
that define what happens in the guest module when the service/method is called by the host application. The code in the brackets
of the `define` statement is in the guest language (in this case, ruby).

## Types
A type can be `native` or `wrapper`. A `wrapper` wraps a type defined in the guest language. A normal  wrapper exposes no operations of the original type. 
This means that wrappers, in this case, will be used just as references in the host language. The keyword `of` separates the name of the wrapper and 
the name of the wrapped type.

## Architecture
mnemo is the core application that can be extended through plugins for different languages. The core application is able to parse 
mnif and minm files, but, depending on the `guest` language defined in the mnif file, they are actually handled by a specific plugin.
Imagine that you want to use a library defined in the guest language A in your application developed in your host language B.
Given a mnif file and a mnim file (and assuming that both plugins for A and B exist), mnemo is able to create an importable module
in B that exposes the interface defined in the mnif file.

The importable module is just a wrapper. Everything declared in the mnif file is visible in the host language. The integration is
seemless: no need to add code, it can be used just as a normal native library. When a call to a service of a module
is performed, the importable module runs a HTTP server for the guest module, exposing the defined services. All the objects are
stored there, and the host application will only have access to them as references. 
