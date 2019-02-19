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
 
- old style cast
```
Derived d;
Base *pb = &d; //implicitlu convert Derived* -> Base*
//remember, sometimes the base pointer could hold different value the &d, if this is the case, a offset is calculated runtime to the Derived* pointer to get the correct Base* pointer value
//this means in c++, a single object might have more than one address(e.g., its address when pointed to by a Base* pointer, and its address when pointed by a Derived* pointer. This virtually happens all the time when multiple inheritance is in use. It can happen under single inherience as well)
```

- static cast
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

//cast derived pointer to a base pointer and then invoke function in base class, this wont work 
//because the cast create a new, temporary copy of the base class part of \*this, then invoke 
//onResize on the copy.
```

- dynamic cast: when you want to perfrom derived class operations on what you believe to be a derived class object, but you have only a pointer- or reference-to-base through which to manipulate the object
  - use pointer-to-derived class directly
  - use pointer-to-base object, and provide a no-op virtual function in base class interface and implement what you need in derived class
- Avoid casts whenever practical, especially dynamic_casts in performance-sensitive code. If a design requires casting, try to develop a cast-free alternative.
- When casting is necessary, try to hide it inside a function. Clients can then call the function instead of putting casts in their own code.
- Prefer C++-style casts to old-style casts. They are easier to see, and they are more specific about what they do.

3. Avoid returning handles(references, pointers, or iterators) to object internals.
```
class Point {                      // class for representing points
public:
  Point(int x, int y);
  ...

  void setX(int newVal);
  void setY(int newVal);
  ...
};

class Rectangle {
public:
  ...
  const Point& upperLeft() const { return pData->ulhc; }
  const Point& lowerRight() const { return pData->lrhc; }
  ...
};

class GuiObject{...};
const Rectangle boundingBox(const GUIObject* obj);
GuiObject *pgo;
const Point *pUpperLeft = &(boundingBox(*pgo).upperLeft()); //*pUpperLeft is dangling

```

4. Strive for exception-safe code


