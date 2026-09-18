# Slides
![[Back-to-Basics-Value-Semantics-Klaus-Iglberger-CppCon-2022.pdf]]
# Modern Visitor Pattern
* Avoids inheritance - so no extra indirection
* Avoids pointers - uses values
# Design of the STL
* Vectors
	* 3 pointers on the stack; begin, end, & capacity
* all containers do a deep copy 
* `const`means `const`
	* `std::vector<int> const x = {1, 2};`
* `std::optional`
	* example: string to in where we want to convey error conditions
	* `std::expected` : C++23
	* 