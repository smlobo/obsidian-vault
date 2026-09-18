## Rule
1. apply to type to the left
2. unless const is extreme left, then it applies to the type to the right
## `const`

### variable
```
const int y = 10;
```
### pointer variables
1. pointer to a `const` value
	1. ```
	   int x{10};
	   char y{'M'};
	   const int* i = &x;
	   const int* j = &y;
	   x = 9;
	   y = 'P';
	   // *i = 6;
	   // *j = 'Q';
	   ```
2. const pointer to a non-const value
	1. ```
	   int x{10};
	   int z{100};
	   int* const i = &x; // constant ptr
	   *i = 6;
	   // i = &z;
	   ```
3. const pointer to a const value
	1. ```
	   int x{9};
	   int z{19};
	   const int* const i = &x;
	   // *i = 6;
	   // i = &z;
	   ```
const arguments in functions
```
void foo(int* y) {...}
int z{8};
const int* i = &z;
// foo(i);
int* const j = &z;
foo(j);
```
### const functions
```
void foo() const {} // no effect
```
### const member function
```
class Test {
	int v;
	int getV() const {
		// v = 10;
		return v;
	}
};
```
### const function parameters
```
void foo(const int x) {...} // cannot modify x
```