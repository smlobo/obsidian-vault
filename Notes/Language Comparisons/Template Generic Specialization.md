||C++|Go|Java|Python|
|---|---|---|---|---|
|Generic mechanism|Templates|Generics|Generics|`TypeVar` / `Generic`|
|Type checking|Compile time|Compile time|Compile time|Usually external (`mypy`, pyright, etc.)|
|Generic specialization|Compile time|Compile time|Usually erased|None|
|Generic bytecode/code per `T`|Yes|Often shape-based|No|**No**|
|Runtime representation|Specialized|Shape/dictionary|Erased reference types|Dynamic Python objects|
|Bytecode generation|LLVM IR → native|SSA → native|JVM bytecode|Python bytecode|
|JIT normally involved|No|No|Yes, JVM|No (CPython)|
|`T` affects generated code|Yes|Yes|Usually no|No|

```
   C++             Go             Java            Python
    │               │              │                │
    ▼               ▼              ▼                ▼
compile-time     compile-time    compile-time     runtime
specialization   shape-based     type checking    dynamic
                 generation      + erasure         dispatch
    │               │              │                │
    ▼               ▼              ▼                ▼
LLVM/native      native         JVM bytecode      bytecode
                                   │                │
                                   ▼                ▼
                                  JIT              JIT*
```

# C++
```
C++
────────────────────────────────────

template<T>
     │
     ▼
  Clang AST
     │
     ▼
instantiate T
     │
     ├── Foo<int>
     ├── Foo<float>
     └── Foo<MyType>
          │
          ▼
       LLVM IR
          │
          ▼
      machine code
```
# Go
```
Go
────────────────────────────────────

func[T]
     │
     ▼
 type checking
     │
     ▼
GC shape / dictionary
     │
     ▼
generic SSA
     │
     ▼
machine code
```
# Java
```
Java
────────────────────────────────────

<T>
 │
 ▼
javac type checking
 │
 ▼
erasure
 │
 ▼
JVM bytecode
 │
 ▼
JIT + profiling
 │
 ▼
machine code
```
# Python
```
Python / CPython
────────────────────────────────────

TypeVar[T]
     │
     ├─────────────────┐
     │                 │
     ▼                 ▼
CPython compiler    mypy/pyright
     │                 │
     ▼                 ▼
Python bytecode    static type info
     │
     ▼
CPython interpreter
     │
     ▼
runtime operations
```
And if a Python JIT is involved:
```
Python bytecode
      │
      ▼
 runtime profiling
      │
      ▼
 specialized native code
```