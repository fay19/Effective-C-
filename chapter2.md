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
7. Don't emit exceptions in destrutor.
8. Never call virtual functions during construction or destruction. In contruction of derived class object, base class constructor is called first. If a virtual function is called in base class constructor, then the base class virtual function is called rathen the derived class version. It never goes down to derived class version. 
- An object didnt become a derived class object until execution of a derived clalss constructor begins.
- Once a derived class destructor has run, the object's derived class data members assume undefined values. So C++ treats them as if they no longer exists. Upon entry to the base class desctructor, the object becomes a base class object, and all parts of C++ - virtual functions, dynamic_cast, etc., -treat it that way.
9. explicit specifier
- specify that a constructor or conversion function(since c++11) is explicit, that is , it cannot be used for implicit conversions and copy initialization
```
struct A
{
  A(int) {}
  A(int, int) {}
  
  //conversion operator: operator Typename()
  //allow objects of the class type to be used as if they were of type Typename and when they are, they are converted to Typename
  // using the conversion function
  operator bool() const { return true;}
};

struct B
{
  explicit B(int) {}
  explicit B(int, int) {}
  explicit operator bool() const { return true };
};

int main {
  A a1 = 1; //OK: copy initialization selects A::A(int), essentially A a1 = A(1);
  A a2(2); //OK: direct initialization selects A::A(int)
  A a3 {4, 5}; //OK: direct initialization selects A::A(int, int)
  A a4 = {4, 5} //OK: copy initialization selects A::A(int, int)
  A a5 = (A)1; //OK: explicit cast performs static_cast
  if (a1); //OK: operator bool
  bool na1 = a1; //OK: copy initialization selects A::operator bool()
  bool na2 = static_cast<bool>(a1); //OK: static_cast performs direct initialization

// B b1 = 1; //error: copy initialization does not consider explicit constructor
  B b2(1); //OK
  B b3{4, 5}; //OK
// B 4 = {4, 5} //error
  B b5 = (B)1 //OK: explicit cast perform static_cast
  if (b2); //OK
// bool nb1 = b2; //error
  bool nb2 = static_cast<bool>(b2); //OK


}
```
10. assignment opertaor return a reference to *this
11. handle self assignment in opertor=
```
class Bitmap { ... };

class Widget
{
  ...
private:
  Bitmap *pb;                                   
};

//self assignment unsafe, exception unsafe
Widget& Widget::operator=(const Widget& rhs)
{
  delete ph;
  ph = new Widget(*rhs.ob); //is rhs and this are the same object, error happens
  return *this
}

//self-assignment safe, exception unsafe
Widget& Widget::operator=(const Widget& rhs)
{
  if (this == &rhs) return *this;
  delete ph;
  ph = new Widget(*rhs.ob); 
  return *this
}

//self-assignment safe, exception safe
Widget& Widget::operator=(const Widget& rhs)
{
  Bitmap *pOrig = pb;               // remember original pb
  pb = new Bitmap(*rhs.pb);         // point pb to a copy of rhs's bitmap
  delete pOrig;                     // delete the original pb

  return *this;
}

//self-assignment safe, exception safe, copy and swap
class Widget {
  ...
  void swap(Widget& rhs);   // exchange *this's and rhs's data;
  ...                       // see Item 29 for details
};

Widget& Widget::operator=(const Widget& rhs)
{
  Widget temp(rhs);             // make a copy of rhs's data
  swap(temp);                   // swap *this's data with the copy's
  return *this;
}

//pass by value of swap *this data with the copy's
Widget& Widget::operator=(Widget rhs)
{
  swap(rhs);
  return *this;
}
```
12. Copy all parts of an object
- Copying functions should be sure to copy all of an object's data members and all of its base class parts.

- Don't try to implement one of the copying functions in terms of the other. Instead, put common functionality in a third function that both call.
