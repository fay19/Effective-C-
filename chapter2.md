# Constructores, Destructors, and Assignment Operators

1. Default constructor, copy constructor, destructor and copy assignment operator will be generated if they are needed
```
Empty e1; //default constructor, destrutor
Empty e2(e1); //copy construtor
e2 = e1; // copy assignment operator
```
2. Generate destrutor won't be virtual unless it inherits from a base class which declares a virtual destrutor

