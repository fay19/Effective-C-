# Constructores, Destructors, and Assignment Operators

1. Default constructor, copy constructor, destructor and copy assignment operator will be generated if they are needed
```
Empty e1; //default constructor, destrutor
Empty e2(e1); //copy construtor
e2 = e1; // copy assignment operator
```
2. If a constructor is declared, complier won't provide default construtor. 
3. Generate destrutor won't be virtual unless it inherits from a base class which declares a virtual destrutor
4. Compilers may refuse to generate these functions if they are not in proper use. 
- Say if a class hold a reference or const variables, complier could refuse to generate copy assignment operator
```
class Dog {
  std::string& name;
  std::size_t age;
};

Dog p("Persephone", 10);
Dog s("Satch", 2);

p = s; //?? this is illegal, because Dog hold a reference to the name string, c++ doesnt provide a way to 
       //make a reference refer to a different object. Alternatively, the string object that p.name 
       //refers to cannot be modified by this assignement either.
```
- if the base class copy assignment opertator is private, compilers reject implicit copy assignment operators in derived class.

5. All compiler generated function are public. To make a uncopyable object:
 - declare copy constructor and copy assignment operator as private
 - delete them
 ```
 Object(const Object&) = delete;
 Object& operator=(const Object&) = delete;
 ```
6. Declare destrutors virtual in polymorphic base classes. When a derived class object is deleted through a pointer to a base class with a non-virtual destructor, results are undefined. Any class with virtual functions should almost certainly have a virtual destrutor
7. 

