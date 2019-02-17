## Design and Declarations

1. Pass by reference to const rather than pass by value
- more efficient
- avoid slicing problem when dealing with polymorphism class(when a derived class object is passed(by value) as a base class object)
- for built-in type and STL iterator and function object types, pass-by-value is usually appropriate.

2. Don't try to return a reference when you must return an object 
3. Delare data members private
4. Prefer non-member non-friend functions to member functions
- object-oriented principles dictate that data should be as encapsulated as possible
```
class WebBrowser {
public:
  ...
  void clearCache();
  void clearHistory();
  void removeCookies();
  ...
};

class WebBrowser {
public:
  ...
  void clearEverything();               // calls clearCache, clearHistory,
                                        // and removeCookies
  ...
};

//this non-member function actally yeilds better encapsulation
void clearBrowser(WebBrowser& wb)
{
  wb.clearCache();
  wb.clearHistory();
  wb.removeCookies();
}
```
5. declare non-member functions with type conversions should apply to all parameters
```
class Rational {
public:
...

const Rational operator*(const Rational& rhs) const;
};

Rational oneEighth(1, 8);
Rational oneHalf(1, 2);

Rational result = oneHalf * oneEighth;            // fine

result = result * oneEighth;                      // fine

result = oneHalf * 2;  // fine with non-explicit c-tot, essentially: result = oneHalf.operator*(2)

result = 2 * oneHalf;  // error! essentially: result = 2.operator*(oneHalf), 
                       // interget 2 has no associated class, hence no operator8 member function

//make it a non-member function
class Rational {

  ...                                             // contains no operator*
};
const Rational operator*(const Rational& lhs,     // now a non-member
                         const Rational& rhs)     // function
{
  return Rational(lhs.numerator() * rhs.numerator(),
                  lhs.denominator() * rhs.denominator());
}
Rational oneFourth(1, 4);
Rational result;

result = oneFourth * 2;                           // fine
result = 2 * oneFourth;                           // hooray, it works!

```

5. Consider support for a non-throwing swap
- swap the values of two objects is to give each the other's value
- std::swap function involves copy 3 objects which is costly
- "pimpl idiom" (pointer to implementation idiom)
```
class WidgetImpl {                          // class for Widget data;
public:                                     // details are unimportant
  ...

private:
  int a, b, c;                              // possibly lots of data —
  std::vector<double> v;                    // expensive to copy!
  ...
};

class Widget {                              // class using the pimpl idiom
public:
  Widget(const Widget& rhs);

  Widget& operator=(const Widget& rhs)      // to copy a Widget, copy its
  {                                         // WidgetImpl object. For
   ...                                      // details on implementing
   *pImpl = *(rhs.pImpl);                    // operator= in general,
   ...                                       // see Items 10, 11, and 12.
  }
  ...

private:
  WidgetImpl *pImpl;                         // ptr to object with this
};                                           // Widget's data
```
- to swap the value of two widget objects, all we really need to do is swap their pimpl pointers
```
namespace std {

  template<>                            // this is a specialized version
  void swap<Widget>(Widget& a,          // of std::swap for when T is
                    Widget& b)          // Widget; this won't compile
  {
    swap(a.pImpl, b.pImpl);             // to swap Widgets, just swap
  }                                     // their pImpl pointers
}
//this won't compile though because pImpl is private data members
```
- declare a member function called swap to do actually swapping and them specialize std::swap to call the member function
```
class Widget {                     // same as above, except for the
public:                            // addition of the swap mem func
  ...
  void swap(Widget& other)
  {
    using std::swap;               // the need for this declaration
                                   // is explained later in this Item

    swap(pImpl, other.pImpl);      // to swap Widgets, swap their
  }                                // pImpl pointers
  ...
};

namespace std {

  template<>                       // revised specialization of
  void swap<Widget>(Widget& a,     // std::swap
                    Widget& b)
  {
    a.swap(b);                     // to swap Widgets, call their
  }                                // swap member function
}
```
- what if Widget and WidgetImpl are class templates instead of classes
```
namespace std {
  template<typename T>
  void swap<Widget<T> >(Widget<T>& a,      // error! illegal code!
                        Widget<T>& b)
  { a.swap(b); }
}
//illegal code, c++ allows partial specialization of class templates, but it doesnt allow it for function template(std::swap)
```
- to "partially specialize" a function template, the usual approach is to add ab overload
```
namespace std {

  template<typename T>             // an overloading of std::swap
  void swap(Widget<T>& a,          // (note the lack of "<...>" after
            Widget<T>& b)          // "swap"), but see below for
  { a.swap(b); }                   // why this isn't valid code
}
// this is illegal as well, because std is special namespace. It's okay to specialize templates in std, but it's not okay to add new templates(or classes or functions or anything else) to std
```
- what's the correct way to do
provide a non-member function swap within the same namespace of the object(not std)
```
namespace WidgetStuff {
  ...                                     // templatized WidgetImpl, etc.

  template<typename T>                    // as before, including the swap
  class Widget { ... };                   // member function

  ...

  template<typename T>                    // non-member swap function;
  void swap(Widget<T>& a,                 // not part of the std namespace
            Widget<T>& b)                                         
  {
    a.swap(b);
  }

//usecase
template<typename T>
void doSomething(T& obj1, T& obj2)
{
  using std::swap;           // make std::swap available in this function
  ...
  swap(obj1, obj2);          // call the best swap for objects of type T
  ...
}

//When compilers see the call to swap, they search for the right swap to invoke. C++'s name lookup rules ensure that this will find any //T-specific swap at global scope or in the same namespace as the type T. (For example, if T is Widget in the namespace WidgetStuff, //compilers will use argument-dependent lookup to find swap in WidgetStuff.) If no T-specific swap exists, compilers will use swap in //std, thanks to the using declaration that makes std::swap visible in this function. Even then, however, compilers will prefer a T-
//specific specialization of std::swap over the general template, so if std::swap has been specialized for T, the specialized version //will be used.

}
```
- to summary, if the default swap isnt efficient enough, do the following:
  - Offer a public swap member function that efficiently swaps the value of two objects of your type. For reasons I'll explain in a moment, this function should never throw an exception.
  - Offer a non-member swap in the same namespace as your class or template. Have it call your swap member function.
  - If you're writing a class (not a class template), specialize std::swap for your class. Have it also call your swap member function.
  - Finally, if you're calling swap, be sure to include a using declaration to make std::swap visible in your function, then call swap without any namespace qualification.
- never throw excepetion in swap. Because highly efficient swaps are almost always based on operations on built-in types (such as the pointers underlying the pimpl idiom), and operations on built-in types never throw exceptions.




