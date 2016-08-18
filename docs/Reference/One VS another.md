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


