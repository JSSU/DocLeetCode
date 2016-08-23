# Function or predefine items

## A

### Access Specifiers
`public` members can be accessed from outside the class.  
`protected` members can be accessed in the class and its derived classes &  
`private` members can be only accessed within the class. 

>Note that the member being `static` or not the same Access Specification rules
> apply to it.
`static` implies the storage specification and that the some member will be shared 
across all the instances of the class it does not change where the member can be 
accessed.

## B

## C

## D

## E

## F

### Functor

 A functor is pretty much just a class which defines the operator(). 
 That lets you create objects which "look like" a function:

 ```
 // this is a functor
struct add_x {
  add_x(int x) : x(x) {}
  int operator()(int y) const { return x + y; }

private:
  int x;
};

// Now you can use it like this:
add_x add42(42); // create an instance of the functor class
int i = add42(8); // and "call" it
assert(i == 50); // and it added 42 to its argument

std::vector<int> in; // assume this contains a bunch of values)
std::vector<int> out(in.size());
// Pass a functor to std::transform, which calls the functor on every element 
// in the input sequence, and stores the result to the output sequence
std::transform(in.begin(), in.end(), out.begin(), add_x(1)); 
assert(out[i] == in[i] + 1); // for all i
```

There are a couple of nice things about functors. One is that unlike regular functions, 
they can contain state. The above example creates a function which adds 42 to whatever 
you give it. But that value 42 is not hardcoded, it was specified as a constructor 
argument when we created our functor instance. I could create another adder, which 
added 27, just by calling the constructor with a different value. This makes them 
nicely customizable.  

As the last lines show, you often pass functors as arguments to other functions such 
as std::transform or the other standard library algorithms. You could do the same with 
a regular function pointer except, as I said above, functors can be "customized" because 
they contain state, making them more flexible (If I wanted to use a function pointer, 
I'd have to write a function which added exactly 1 to its argument. The functor is 
general, and adds whatever you initialized it with), and they are also potentially more 
efficient. In the above example, the compiler knows exactly which function `std::transform` 
should call. It should call `add_x::operator()`. That means it can inline that function call. 
And that makes it just as efficient as if I had manually called the function on each value 
of the vector.  

If I had passed a function pointer instead, the compiler couldn't immediately see which 
function it points to, so unless it performs some fairly complex global optimizations, it'd 
have to dereference the pointer at runtime, and then make the call.  


## G

## H

### heap

#### make_heap()

```cpp
// range heap example
#include <iostream>     // std::cout
#include <algorithm>    // std::make_heap, std::pop_heap, std::push_heap, std::sort_heap
#include <vector>       // std::vector

using namespace std;
int main () {
  //int myints[] = {10,20,30,5,15};
  //std::vector<int> v(myints,myints+5);

  vector<int> v{10,20,30,5,15};
  std::make_heap (v.begin(),v.end());
  std::cout << "initial max heap   : " << v.front() << '\n';
  std::cout << "v range :";
  for (unsigned i=0; i<v.size(); i++)
    std::cout << ' ' << v[i];   std::cout << '\n';

  std::pop_heap (v.begin(),v.end()); v.pop_back();
  std::cout << "max heap after pop : " << v.front() << '\n';
    std::cout << "v range :";
  for (unsigned i=0; i<v.size(); i++)
    std::cout << ' ' << v[i];   std::cout << '\n';
//---------------------------------------------
  v.pop_back(); std::pop_heap (v.begin(),v.end()); 
  std::cout << "max heap after pop : " << v.front() << '\n';
    std::cout << "v :";
  for (unsigned i=0; i<v.size(); i++)
    std::cout << ' ' << v[i];   std::cout << '\n';



  v.push_back(99); std::push_heap (v.begin(),v.end());
  std::cout << "max heap after push: " << v.front() << '\n';
  std::cout << "final sorted range :";
  for (unsigned i=0; i<v.size(); i++)
    std::cout << ' ' << v[i];   std::cout << '\n';
    
  std::sort_heap (v.begin(),v.end());
 //make_heap(v.begin(),v.end());
  std::cout << "final sorted range :";
  for (unsigned i=0; i<v.size(); i++)
    std::cout << ' ' << v[i];

  std::cout << '\n';

  return 0;
}
```
#### result
    initial max heap   : 30
    v range : 30 20 10 5 15
    max heap after pop : 20
    v range : 20 15 10 5
    max heap after pop : 15
    v : 15 10 20
    max heap after push: 99
    final sorted range : 99 15 20 10
    final sorted range : 10 15 20 99


## I 

## J

## K

## L

### Lambda functions
Constructs a closure: an unnamed function object capable of capturing variables in scope.
Syntax

[ `capture-list` ] ( `params` ) mutable(optional) constexpr(optional)(c++17) exception attribute -> ret { body }  (1)  
[ `capture-list` ] ( `params` ) -> ret { `body` } (2)   
[ `capture-list` ] ( `params` ) { `body` }  (3)   
[ `capture-list` ] { `body` } (4)   
1) Full declaration.
2) Declaration of a const lambda: the objects captured by copy cannot be modified.
3) Omitted trailing-return-type: the return type of the closure's operator() is determined according to the following rules:


* if the body consists of nothing but a single return statement with an expression, 
the return type is the type of the returned expression 
(after lvalue-to-rvalue, array-to-pointer, or function-to-pointer 
implicit conversion);
* otherwise, the return type is void.
(until C++14)  
* The return type is deduced from return statements as if for a 
function whose return type is declared auto.
4) Omitted parameter list: function takes no arguments, as if the parameter list was ().

Explanation  
`mutable`	-	allows body to modify the parameters captured by copy, and to call their non-const member functions  
`constexpr(C++17)`	-	explicitly specifies that the function call operator is a constexpr function. When this specifier is not present, the function call operator will be constexpr anyway, if it happens to satisfy all constexpr function requirements  
`exception`	-	provides the exception specification or the noexcept clause for operator() of the closure type  
`attribute`	-	provides the attribute specification for operator() of the closure type  
`capture-list`	-	a comma-separated list of zero or more captures, optionally beginning with a capture-default.  
Capture list can be passed as follows (see below for the detailed description):    
+ **[a,&b]** where a is captured by value and b is captured by reference.  
+ **[this]** captures the this pointer by value
+ **[&]** captures all automatic variables odr-used in the body of the lambda by reference
+ **[=]** captures all automatic variables odr-used in the body of the lambda by value
+ **[]** captures nothing  
`params`	-	The list of parameters, as in named functions, except that default arguments are not allowed (until C++14). If auto is used as a type of a parameter, the lambda is a generic lambda. (since C++14)  
`ret`	-	Return type. If not present it's implied by the function return statements (or void if it doesn't return any value)  
`body`	-	Function body
The lambda expression is a prvalue expression whose value is 
`(until C++17)`whose result object is `(since C++17)` an unnamed
 temporary object of unique unnamed non-union non-aggregate class
 type, known as closure type, which is declared (for the purposes 
 of ADL) in the smallest block scope, class scope, or namespace 
 scope that contains the lambda expression. The closure type has 
 the following members:...
 ```cpp
 int x = 4;
 auto y = [&r = x, x = x+1]()->int {
            r += 2;
            return x+2;
         }();  // Updates ::x to 6, and initializes y to 7.
```

## M

## N

## O 

## P

## Q

## R

## S

## T

## U

### unordered_map
#### Syntax:
- [key] = value
- .insert
- .erase 

## V

### vector<>
#### Syntax:
- .emplace_back(`the primeters`)



## W

## X

## Y 

## Z







