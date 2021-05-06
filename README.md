<div align='center'>
    <h1>CS2433: POPL2</h1>
    <h2>
        Mini Assignment 2 : Large Codebases and OOP
    </h2>
    <em>Github: https://github.com/dark-trojan</em>
</div>

​							

### Overview of Clang 

The Clang compiler has three phases of parsing source code:

- The **front-end**: The front-end does the actual parsing of source code, while checking for compile-time errors; if there aren't any, it builds a language-specific Abstract Syntax Tree (AST) that corresponds to the given input code.
- The **optimizer**: As the name suggests, the optimizer "optimizes" the AST, before it is processed further, to enable quicker and more efficient execution of the finally generated executable file. 
- The **back-end** : This finally generates the executable code which is to be executed by the host machine.

The USP of Clang is its utilization of LLVM; LLVM makes use of LLVM Intermediate Representation (LLVM IR), which can be thought of as similar to Java Byte-code.
LLVM IR is designed to host analyses and transformations in the optimizer section of a compiler. It enables lightweight run-time optimizations, cross-function optimizations, whole program analysis, etc.

------

### C++11/C++14 Features

In case the attached images don't load due to any bug, the associated file names and line numbers are provided as well.

**constexpr**: Constant expressions are expressions evaluated by the compiler at compile-time. Only non-complex computations can be carried out in a constant expression. The *constexpr* specifier is used to indicate this.

![image-20210406233129212](/home/darktrojan/snap/typora/33/.config/Typora/typora-user-images/image-20210406233129212.png)

The above code is found in `llvm/include/llvm/ADT/Bitfields.h, Line 106` and is used to instantiate bitsets of various types.



**auto**: The *auto* keyword tells the compiler to perform automatic type-inference, and was introduced in C++11. It is used plenty of times in the code-base, most commonly to declare iterators, such as the example given below, from `llvm/lib/Frontend/OpenMP/OMPContext.cpp, Line 106`.

![image-20210406233217681](/home/darktrojan/snap/typora/33/.config/Typora/typora-user-images/image-20210406233217681.png)

Here, C0 and C1 are of type ArrayRef, which is a custom data structure defined in LLVM. It is used to represent a constant reference to an array.



**Range-based for loops**: Introduced in C++11, range-based for loops provide a method to directly iterate over the elements of a given data structure / container. For example, the below code can be found in `llvm/lib/Transforms/Scalar/LoopUnrollPass.cpp, Line 1423`, and is an important intermediary step in the unrolling of loops. If not for this functionality, the associated work to achieve the same functionality would be extra; for example, we might have to pass the length of the container to the function to iterate over the required range.

![image-20210406233235362](/home/darktrojan/snap/typora/33/.config/Typora/typora-user-images/image-20210406233235362.png)



**Right-angle brackets**: A simplistic design change, which allows the programmer to use right angle brackets without spaces while declaring containers. For example, the following snippet of code can be found in `llvm/lib/AsmParser/LLParser.h, Line 150`. 

![image-20210406233245862](/home/darktrojan/snap/typora/33/.config/Typora/typora-user-images/image-20210406233245862.png)

Previously, the same code would have been of the form `<...LocTy> > > ForwardRefTypeIDs;`. 



**Strongly typed enums**: Introduced in C++11, type-safe enums solve a variety of problems with C-style enums including implicit conversions, inability to specify the underlying type and scope pollution. The following code snippet can be found in `flang/runtime/type-info.h, Line 77`. Flang is the Clang equivalent for FORTRAN (LLVM IR allows the usage of a common optimizer for multiple front-ends)

![image-20210406233301042](/home/darktrojan/snap/typora/33/.config/Typora/typora-user-images/image-20210406233301042.png)



**Generic Lambda expressions:** Lambda expressions were introduced in C++ 11; essentially inline functions, a *lambda* is an unnamed function object capable of capturing variables in scope. This concept was extended upon in C++14, with generic lambdas. Now, the programmer can use the *auto* type-specifier in the parameter list, enabling polymorphic lambdas. Generic lambdas can be observed in the code-base, such as in `llvm/unittests/ADT/TypeSwitchTest.cpp, Line 60` as seen below.

![image-20210406233315154](/home/darktrojan/snap/typora/33/.config/Typora/typora-user-images/image-20210406233315154.png)

------

### Class Hierarchy

LLVM is a massive code-base, with an extraordinary number of classes. Thus, it is important to properly organize these classes into classes and sub-classes, and define a proper inheritance structure. As mentioned, LLVM is a giant code-base, and it would be impossible to capture this entire hierarchy in the matter of a page or two. Instead, kindly refer to the following link to observe its class hierarchy:

[LLVM Class Hierarchy](https://llvm.org/doxygen/hierarchy.html)

A more visual representation would be to represent the hierarchy as a directed graph. This graphical form can be observed in the link below:

[LLVM Graphical Class Hierarchy](https://llvm.org/doxygen/inherits.html) 

------

### Design Patterns & OOP Design Decisions

*All data obtained from 'Lessons to learn from the Clang/LLVM codebase', CppDepend Blog*

- LLVM code is highly **modular**. The modularity is achieved in the following ways:
  - **Libraries**: The different methods in the code-base are organized into different libraries. This is neat, encourages the use of good interfaces and makes the introduction gentler for new developers (as they only need to focus on the library corresponding to their task).
  - **Classes**: The data and methods are abstracted into classes, nested classes and derived classes as much as possible. In fact, there are more than 6000 derived classes in the codebase. This encourages object oriented programming, which is modular in and of itself.
  - **Namespaces**: There are more than 500 namespaces in the codebase; this provides modularity and makes the code readable and maintainable.
- **Objected Oriented Programming**:
  - **Inheritance**: As mentioned earlier, inheritance is widely prevalent in the codebase, which encourages reusability of code without repetition. The inheritance graph can be observed in the class hierarchy linked above.
  - **Virtual Methods**: The presence of virtual methods, especially pure virtual methods, allows derived classes to flexibly extend the base class, while maintaining a basic structure. There are more than 3000 virtual methods, of which more than 1000 are pure virtual, in the Clang/LLVM codebase.
- **Generic Programming**: More than 1000 generic types (of which a majority are generic methods) are present in the codebase. The mechanism for instantiating C ++ templates ensures that when generalized algorithms and data structures are used, a fully optimized and specialized version will be created specifically for specific parameters, allowing generalized algorithms to be as effective as their non-generalized versions.
- **Clean Code**: 
  - **Side Effects:** In programming a *side effect* is when a procedure changes a variable from outside its scope. One way to increase the reliability of a module is to get rid of side effects. This makes compiling and integrating modules easier and more reliable. More than 100,000 functions in the codebase have no side effects.
  - **Implementation:** Less than 2% of the functions have a definition larger than 60 lines, only 500 functions take more than 8 parameters as arguments, and less than 1% of the functions have greater than 25 local variables.

------

### Custom Data Structures

LLVM implements its own version of different data structures to improve performance. The entire list of all such data structures can be found in the `llvm/include/llvm/ADT` directory, or in this [link](https://llvm.org/doxygen/dir_32453792af2ba70c54e3ccae3a790d1b.html). Some relevant examples are:

- **llvm::SmallVector**
  - Similar to std::vector in both functionality and interface
  - Optimized to maintain a fixed number of elements
  - Also flexible to add more elements if needed

- **llvm::DenseMap**
  - [Quadratically probed](https://en.wikipedia.org/wiki/Quadratic_probing) HashMap.
  - Keeps everything in a single memory allocation
  - Similar to std::map's interface
    - insert(std::pair<>)
    - find(Key&)
    - count(Key&)
    - begin(), end()
  - **llvm::StringMap** is similar, but defined solely for strings.

- **llvm::SmallSet**
  - Used as a substitute for std::set
  - Implemented as a small, unsorted vector
  - Searches take linear time
  - The data structure is only efficient for small set sizes. After a certain threshold, it is expanded to an std::set to maintain reasonable lookup times.
