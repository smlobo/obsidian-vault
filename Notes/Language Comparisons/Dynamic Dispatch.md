||C++|Go|Java|Python|
|---|---|---|---|---|
|Primary mechanism|`virtual` functions|Interfaces|Virtual/interface methods|Attribute lookup + descriptors|
|Dispatch target known at compile time?|Sometimes|Sometimes|Sometimes|Usually no|
|Typical runtime mechanism|vtable|itable/interface table|vtable / itable|`__getattribute__`, descriptors, type lookup|
|Object carries type info?|Usually via vptr for polymorphic objects|Interface value carries type info|Object has class pointer|Every object has `PyTypeObject*`|
|Can compiler devirtualize?|Yes|Yes|Yes, especially JIT|JIT/interpreter can specialize|
|Runtime type checking|Limited|Interface assertions|Built in|Fundamental to execution|
|Multiple inheritance|Yes|No|No class MI|Flexible object model|

```
C++:
"Which implementation does the C++ type system say this object uses?"
        ↓
vtable if necessary


Go:
"Which concrete type is stored in this interface?"
        ↓
itab/interface dispatch


Java:
"Which implementation belongs to this object's runtime class?"
        ↓
JVM virtual/interface dispatch
        ↓
JIT may eliminate it


Python:
"What does the attribute 'foo' mean on this object right now?"
        ↓
Python attribute lookup
        ↓
descriptor/MRO/dictionary machinery
        ↓
call
```
# C++
```
object
  ↓
vptr
  ↓
vtable[index]
  ↓
function
```
# Go
```
interface value
  ↓
itab/type information
  ↓
method slot
  ↓
function
```
# Java
```
object
  ↓
class metadata
  ↓
virtual method resolution
  ↓
function
```
# Python
```
object
  ↓
attribute lookup
  ↓
MRO / dictionaries / descriptors
  ↓
callable
```
