## Templates and Generic Programming

1. OOD is resolved around explicit interface and run-time polymorphism.
2. Template is more about implicit interface and compile-time polymorphism.
- implicit interface: a set of valid expressions
3. Precede it with word typename when refer a nested dependent type name in a template
```
template<typename C>
void food(const C& container) {
  typename C::const_iterator iter;
  ... 
}
```

4. In derived class templates, refer to names in base class templates via a “this->” prefix, via using declarations, or via an explicit base class qualification.

```
class CompanyA {
public:
  ...
  void sendCleartext(const std::string& msg);
  void sendEncrypted(const std::string& msg);
  ...
};

class CompanyB {
public:
  ...
  void sendCleartext(const std::string& msg);
  void sendEncrypted(const std::string& msg);
  ...
};
...                                     // classes for other companies

class MsgInfo { ... };                  // class for holding information
                                        // used to create a message
template<typename Company>
class MsgSender {
public:
  ...                                   // ctors, dtor, etc.

  void sendClear(const MsgInfo& info)
  {
    std::string msg;
    create msg from info;

    Company c;
    c.sendCleartext(msg);
  }

  void sendSecret(const MsgInfo& info)   // similar to sendClear, except
  { ... }                                // calls c.sendEncrypted
};

template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
  ...                                    // ctors, dtor, etc.
  void sendClearMsg(const MsgInfo& info)
  {
    write "before sending" info to the log;

    sendClear(info);                     // call base class function, this code will not compile!
    write "after sending" info to the log;
  }
  ...

};

//The problem is that when compilers encounter the definition for the class template LoggingMsgSender, they don't know what class it //inherits from. Sure, it's MsgSender<Company>, but Company is a template parameter, one that won't be known until later (when //LoggingMsgSender is instantiated). Without knowing what Company is, there's no way to know what the class MsgSender<Company> looks //like. In particular, there's no way to know if it has a sendClear function

//3 solulutions

//this->
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
  ...                                    // ctors, dtor, etc.
  void sendClearMsg(const MsgInfo& info)
  {
    write "before sending" info to the log;

    this->sendClear(info); // okay, assumen that sendclear will be inherited
    write "after sending" info to the log;
  }
  ...

};

//using declaration
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
  using MsgSender<Company>::sendClear; //tell compilers to assume that sendClear is in the base class
  ...                                    // ctors, dtor, etc.
  void sendClearMsg(const MsgInfo& info)
  {
    write "before sending" info to the log;

    sendClear(info); // okay, assumes that sendClear will be inherited
    write "after sending" info to the log;
  }
  ...

};

//explicit base class qualification
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
  ...
  void sendClearMsg(const MsgInfo& info)
  {
    ...
    MsgSender<Company>::sendClear(info);      // okay, assumes that
    ...                                       // sendClear will be
  }                                           //inherited

  ...
};


```
5. template specializetion
- "template <>" at the beginning of class definition. It signifies that this is neither a template or a standalone class. Rather, it is a specialized version of the MsgSender template to be used when the template argument is CompanyZ.This is known as a total template specialization: the template MsgSender is specialized for the type CompanyZ, and the specialization is total — once the type parameter has been defined to be CompanyZ, no other aspect of the template's parameters can vary.
```
class CompanyZ {                             // this class offers no
public:                                      // sendCleartext function
  ...
  void sendEncrypted(const std::string& msg);
  ...
};

//general MsgSender template is inappropriate for companyZThis is known as a total template specialization: the template MsgSender is specialized for the type CompanyZ, and the specialization is total — once the type parameter has been defined to be CompanyZ, no other aspect of the template's parameters can vary.
template<>                                 // a total specialization of
class MsgSender<CompanyZ> {                // MsgSender; the same as the
public:                                    // general template, except
  ...                                      // sendCleartext is omitted
  void sendSecret(const MsgInfo& info)
  { ... }
};
```

6. Use member function templates to accept "all compatible types"

- look at the case of smart pointer, thin about how to make the following code compile
```
class Top { ... };
class Middle: public Top { ... };
class Bottom: public Middle { ... };
//compliable since raw pointers has implicit type conversion
Top *pt1 = new Middle;                   // convert Middle* ⇒Top*
Top *pt2 = new Bottom;                   // convert Bottom* ⇒Top*
const Top *pct2 = pt1;                   // convert Top* ⇒ const Top*

//cannot compile, because there is no inherent relationship among different instantiations of the same template, so compliers view 
//SmartPtr<Middle> and SmartPtr<Top> as completely different classes.
template<typename T>
class SmartPtr {
public:                             // smart pointers are typically
  explicit SmartPtr(T *realPtr);    // initialized by built-in pointers
  ...
};

SmartPtr<Top> pt1 =                 // convert SmartPtr<Middle> ⇒
  SmartPtr<Middle>(new Middle);     //   SmartPtr<Top>

SmartPtr<Top> pt2 =                 // convert SmartPtr<Bottom> ⇒
  SmartPtr<Bottom>(new Bottom);     //   SmartPtr<Top>

SmartPtr<const Top> pct2 = pt1;     // convert SmartPtr<Top> ⇒
                                    //  SmartPtr<const Top>
```
- a construtor template is needed, such templates are example of member function templates/member templates - templates that generate member functions of a class
```
template<typename T>
class SmartPtr {
public:
  template<typename U>                       // member template
  SmartPtr(const SmartPtr<U>& other);        // for a "generalized
  ...                                        // copy constructor"
};
//note The generalized copy constructor above is not declared explicit. That's deliberate. Type conversions among built-in pointer types //(e.g., from derived to base class pointers) are implicit and require no cast, so it's reasonable for smart pointers to emulate that //behavior. Omitting explicit on the templatized constructor does just that.
```
7. implicit type conversion functions are never considered during template argument deduction
```
template<typename T>
class Rational {
public:
  Rational(const T& numerator = 0,     // see Item 20 for why params
           const T& denominator = 1);  // are now passed by reference

  const T numerator() const;           // see Item 28 for why return
  const T denominator() const;         // values are still passed by value,
  ...                                  // Item 3 for why they're const
};

template<typename T>
const Rational<T> operator*(const Rational<T>& lhs,
                            const Rational<T>& rhs)
{ ... }

Rational<int> oneHalf(1, 2);          // this example is from Item 24,
                                      // except Rational is now a template

Rational<int> result = oneHalf * 2;   // error! won't compile
```
- Solution: frient function, because a friend function declaration in a template can refer to a specific function, it doesnt depend on template argument deduction
```
template<typename T>
class Rational {
public:
  ...

friend                                              // declare operator*
  const Rational operator*(const Rational& lhs,     // function (see
                           const Rational& rhs);    // below for details)
};

template<typename T>                                // define operator*
const Rational<T> operator*(const Rational<T>& lhs, // functions
                            const Rational<T>& rhs)
{ ... }
//Now our mixed-mode calls to operator* will compile, because when the object oneHalf is declared to be of type Rational<int>, the class //Rational<int> is instantiated, and as part of that process, the friend function operator* that takes Rational<int> parameters is //automatically declared. As a declared function (not a function template), compilers can use implicit conversion functions (such as //Rational's non-explicit constructor) when calling it, and that's how they make the mixed-mode call succeed.
```
