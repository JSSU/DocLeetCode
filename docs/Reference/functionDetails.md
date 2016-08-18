# Function or predefine items

## Access Specifiers
`public` members can be accessed from outside the class.  
`protected` members can be accessed in the class and its derived classes &  
`private` members can be only accessed within the class. 

>Note that the member being `static` or not the same Access Specification rules
> apply to it.
`static` implies the storage specification and that the some member will be shared 
across all the instances of the class it does not change where the member can be 
accessed.


## Heap

### make_heap()

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
