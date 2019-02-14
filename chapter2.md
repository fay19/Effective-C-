# Constructores, Destructors, and Assignment Operators

1. Default constructor, copy constructor, destructor and copy assignment operator will be generated if they are needed
```
Empty e1; //default constructor, destrutor
Empty e2(e1); //copy construtor
e2 = e1; // copy assignment operator
```
2. Generate destrutor won't be virtual unless it inherits from a base class which declares a virtual destrutor
3. Compilers may refuse to generate these functions if they are not in proper use. Say if a class hold a reference or const variables, complier could refuse to generate copy assignment operator
4. All compiler generated function are public. To prevent these function from being generated, declare them yourself! Declare copy construtor and copy assignment operator private if no copy is allowed.
5. Declare destrutors virtual in polymorphic base classes. When a derived class object is deleted through a pointer to a base class with a non-virtual destructor, results are undefined. Any class with virtual functions should almost certainly have a virtual destrutor
6. 

