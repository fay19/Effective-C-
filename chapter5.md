## Implementation
1. Postpone variable definitions as long as possible. It increases program clarity and improves program efficiency.
2. Minimize casting
- old style casting
```
(T) expression //cast expression to be of type T

T(expression) // cast expression to be of type T

//no difference between these two
```
- new cast forms
  - const_cast<T>(expression), typically used to cast away the constness of objects
  - dynamic_cast<T>(expression), primarily used to perform "safe downcasting", the only cast cannot be performed by old-style cast, and the only cast that may have a significant run time cost
  - reinterpret_cast<T>(expression), intended for low-level cast, e.g. casting a pointer to an int
  - static_cast<T>(expression)
    - implicit conversions(e.g., non-const object to const object, int to double, etc.) 
    - void* pointers to typed pointers
    - pointer-to-base to pointer-to-derived
```
Derived d;
Base *pb = &d; //implicitlu convert Derived* -> Base*
//remember, sometimes the base pointer could hold different value the &d, if this is the case, a offset is calculated runtime to the Derived* pointer to get the correct Base* pointer value
//this means in c++, a single object might have more than one address(e.g., its address when pointed to by a Base* pointer, and its address when pointed by a Derived* pointer. This virtually happens all the time when multiple inheritance is in use. It can happen under single inherience as well)
```
```
class Window {                                // base class
public:
  virtual void onResize() { ... }             // base onResize impl
  ...
};

class SpecialWindow: public Window {          // derived class
public:
  virtual void onResize() {                   // derived onResize impl;
    static_cast<Window>(*this).onResize();    // cast *this to Window,
                                              // then call its onResize;
                                              // this doesn't work!
  }
};

//cast derived pointer to a base pointer and then invoke function in base class, this wont work because the cast create a new, temporary copy of the base class part of \*this, then invoke onResize on the copy.
```
