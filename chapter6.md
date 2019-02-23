## Inheritance and Object-Oriented Design

1. Non-virtual functions are statically bound, virtual functions are dynamically bound.
- An object's static type is the type you declare it to have in the program text.
```
Shape *ps;                       // static type = Shape*, static type: pointer to shape
Shape *pc = new Circle;          // static type = Shape*, static type: pointer to shape, dynamic type: pointer to circle
Shape *pr = new Rectangle;       // static type = Shape*
```
- An object's dynamic type is determined by the type of the object to which it currently refers, dynamic type indicates how it will behave.
- virtual functions are dynamically bound, meaning that particular function called is determined by the dynamic type of the object throught which it's invoked

2. Never refine a virtual function that has a default parameter value
- because virtual function are dynamic bound, but default parameter values are statically bound.

3. Private inheritance means is-implemented-in-terms of.
- there is no is-a relationship
- all members inherited from base class are private in derived class

