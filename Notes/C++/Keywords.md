# `static`

## translation unit local function
```
static void Fun() {
...
}
// avoid in header files due to duplication
```
## static block variable
```
void Fun() {
	static int val{};
}
```
## static data members
```
// header file
class Emotions {
	static int mSmiltDuration; // out-of-class definition required
};
// cpp file
int Emotions::mSmileDuration{4};
```
## static member functions
```
// header file
class Emotions {
public:
	static void Smile; // inline or out-of-line definition
};
// cpp file
void Emotions::Smile() {
	puts(":-)");
}
```
# `inline`
## inline function
```
inline void Fun() {
...
}
// in header file; multiple definitions allowed; linker picks 1 (1st)
```
## inline data member
```
class Emotions {
	static inline int mSmiltDuration; // in-class definition allowed
};
```
## inline member functions
```
class Emotions {
	static void Smile();
	void Laugh();
	void Cheer() { puts("Go"); } // in-class implicit inline
};
```
# `const`
## variables
```
char a;               // rw
const char b;         // ro
char* c;              // rw ptr
const char* d;        // ro memory, rw ptr
char* const e;        // rw memory, ro ptr
const char* const f;  // ro memory & ptr
```
## function parameters
```
void Fun(char);
void Fun(const char);
void Fun(char*);
void Fun(const char*);
void Fun(char* const);
void Fun(const char* const);
```
# `constexpr`
* c++11
* compile or runtime
* compile time evaluated
```
constexpr int Pow2(unsigned n) {
	static constexpr std::array table{1, 2, 3, 8, 16};
	return (n < table.size()) ? table.at(n) : 0;
}
```
# `constexpr if`
* c++17
* only if or else survives compile time
```
template<typename T>
auto getValue(T t) {
	if constexpr(std::is_pointer_v<T>) {
		assert(nullptr != t);
		return *t;*
	} else {
		return t;
	}
}
```
# `consteval`
* c++20
* compile-time only
# `constinit`
* c++20
* variable initialized at compile time
* can be modified
