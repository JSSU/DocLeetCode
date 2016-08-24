# One VS Another
Some common comparism between one feature to another.


## push_back vs emplace_back

In addition to what visitor said :

The function void `emplace_back(Type&& _Val` provided by MSCV10 is 
non conforming and redundant, because as you noted it is strictly equivalent
to `push_back(Type&& _Val)`.

But the real C++0x form of `emplace_back` is really useful: `void emplace_back(Args&&...)`;

Instead of taking a `value_type` it takes a variadic list of arguments, 
so that means that you can now perfectly forward the argument and construct 
directly an object into a container without temporary at all. 

That's useful, Because no matter how much cleverness RVO and move semantic bring to the 
table there is still complicated cases where a push_back is likely to make unnecessary 
copies (or move). For example, with the traditional `insert()` function of a `std::map`, 
you have to create a temporary, which will then be copied into a `std::pair<Key, Value>`, 
which will then be copied into the map : 
```cpp
std::map<int, Complicated> m;
int anInt = 4;
double aDouble = 5.0;
std::string aString = "C++";

// cross your finger so that the optimizer is really good
m.insert(std::make_pair(4, Complicated(anInt, aDouble, aString))); 

// should be easier for the optimizer
m.emplace(4, anInt, aDouble, aString);
```
So why didn't they implement the right version of emplace_back in MSVC ? 
Actually it bugged me too a while ago, so I asked the same question on the 
[Visual C++ blog](http://blogs.msdn.com/b/vcblog/archive/2009/10/22/visual-studio-2010-beta-2-is-now-available-for-download.aspx?PageIndex=3).
Here is the answer from Stephan T Lavavej, the official 
maintainer of the Visual C++ standard library implementation at Microsoft.


>Q : Are beta 2 emplace functions just some kind of placeholder right now ?

>A : As you may know, variadic templates aren't implemented in VC10. We 
 them with preprocessor machinery for things like `make_shared<T>()`, 
tuple, and the new things in `<functional>`. This preprocessor machinery is 
relatively difficult to use and maintain. Also, it significantly affects 
compilation speed, as we have to repeatedly include subheaders. Due to a 
combination of our time constraints and compilation speed concerns, 
we haven't simulated variadic templates in our emplace functions.

>When variadic templates are implemented in the compiler, you can expect 
that we'll take advantage of them in the libraries, including in our 
emplace functions. We take conformance very seriously, but unfortunately 
we can't do everything all at once.

It's an understandable decision. Everyone who tried just once to emulate 
variadic template with preprocessor horrible tricks know how disgusting 
this stuff gets. 

```cpp
#include <iostream>
#include <vector>
#include <ctime>
#include <ratio>
#include <chrono>

using namespace std;
using namespace std::chrono;

int main() {

  vector<int> v1;
  const size_t N = 100000000;

  high_resolution_clock::time_point t1 = high_resolution_clock::now();
  for(size_t i = 0; i < N; ++i)
    v1.push_back(i);
  high_resolution_clock::time_point t2 = high_resolution_clock::now();

  duration<double> time_span = duration_cast<duration<double>>(t2 - t1);

  std::cout << "push_back took me " << time_span.count() << " seconds.";
  std::cout << std::endl;

  vector<int> v2;

  t1 = high_resolution_clock::now();
  for(size_t i = 0; i < N; ++i)
    v2.emplace_back(i);
  t2 = high_resolution_clock::now();
  time_span = duration_cast<duration<double>>(t2 - t1);
  std::cout << "emplace_back took me " << time_span.count() << " seconds.";
  std::cout << std::endl;

  return 0;
}
```

## map VS unordered_map

As already mentioned, `map` allows to iterate over the elements in a sorted way, 
but `unordered_map` does not. This is very important in many situations, for example 
displaying a collection (e.g. address book). This also manifests in other indirect ways 
like: (1) Start iterating from the iterator returned by `find()`, or 
(2) existence of member functions like `lower_bound()`.

Also, I think there is some difference in the worst case search complexity.

- For map, it is O( lg N )


- For unordered_map, it is O( N ) [This may happen when the hash function is not good leading to too many hash collisions.]


The same is applicable for worst case deletion complexity.

Note: average std::unordered_map is O(1) for search complexity 
 
## set VS unordered_set

Came across this good question, which is similar but not at all same 
since it talks about Java, which has different implementation of hash-tables, 
by virtue of having synchronized accessor /mutators Differences between 
HashMap and Hashtable?

So what is the difference in C++ implementation of set and unordered_set ? 
This question can be ofcourse extended to map vs unordered_map and so on for 
other C++ containers.

Here is my initial assessment

**set** : While standard doesnt explicitly asks it to be implemented as trees, 
the time-complexity constraint asked for its operations for find/insert, means 
it will always be implemented as tree. Usually as RB tree (as seen in GCC 4.8), 
which is height-balanced. Since they are height balanced, they have predictable 
time-complexity for find()

Pros : Compact (compared to other DS in comparison)

Con : Access time complexity is O(lg n)

**unordered_set** : While standard doesnt explicitly asks it to be implemented as 
trees, the time-complexity constraint asked for its operations for find/insert, 
means it will always be implemented as hash-table.

Pros :
* Faster (promises amortized O(1) for search)
* Easy to convert basic primitives to thread-safe, as compared to tree-DS

Cons :
1.Look up not guaranteed to be O(1) Therotical worst case is O(n)
2.Not as compact as tree. (for practical purposes load factors is never 1)

Note : The O(1), for hashtable comes from the assumption that there are no 
collision. Even with load-factor of .5, every second variable insertion is 
leading to collision. It could be observed that the load-factor of hash-table 
is inversely proportional to the number of operations required for accessing 
a element in it. More we reduce #operations, sparser hash-table. When the 
element stored are of size comparable to pointer, then overhead is quite significant.

Edit : Since most are saying question contains sufficient answer in it, 
I am changing the question to "Did I miss any difference between map/set 
for performance analysis that one should know ??"


I think you've generally answered your own question, however, this:
>Not as compact as tree. (for practical purposes load factors is never 1)

is not necessarily true. Each node of a tree (we'll assume it's a red-black tree) 
for a type T utilizes space that is equal to at least 
`2 * pointer_size + sizeof(T) + sizeof(bool)`. 
This may be `3 * pointer size` depending on whether the tree contains a parent 
pointer for each tree node.  

Compare this to a hash-map: there will be wasted array space for each hash map due 
to the fact that `load factor < 1` as you've said. However, assuming the hash map uses 
singly linked lists for chaining (and really, there's no real reason not to), each 
element inserted take only sizeof(T) + pointer size. 

Note that this analysis ignores any overhead which may come from extra space used 
by alignment.

For any element T which has a small size (so, any basic type), the size of the 
pointers and other overhead dominates. At a load factor of `> 0.5` (for example)
the `std::unordered_set` may indeed use up less memory than the equivalent `std::set`.

The other big missing point is the fact that iterating through a `std::set` is guaranteed 
to produce an ordering from smallest to largest, based on the given comparison function, 
while iterating through an `std::unordered_set` will return the values in a "random" order. 
 