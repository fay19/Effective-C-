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
