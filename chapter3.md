1. Use RCSP(reference counting smart pointers for resource management.
2. Use the same form in corresponding uses of *new and *delete
- new expression 
  - memory is allocated
  - one or more constructors are called for that memory
- delete expression
  - one or more destructors are called for that memory
  - the memory is deallocated
- a big question for delete: how many objects reside in the memory being deleted
  - does the pointer being deleted point to a single object or to an array of objects
```
std::string *stringPtr1 = new std::string;

std::string *stringPtr2 = new std::string[100];
...

delete stringPtr1;                       // delete an object

delete [] stringPtr2;                    // delete an array of objects
```
- If you use [] in a new expression, you must use [] in the corresponding delete expression. If you don't use [] in a new expression, you mustn't use [] in the corresponding delete expression.
3. Store newed objects in smart pointers in standalone statements.
```
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority()); //this could result memory leak
/*
before complier call processWidget, complier must generate code to do these 3 things:
1. call priority
2. execute "new Widget"
3. call tr1::shared_ptr constructor

2 must be executed before 3, but the call to priority can be performed first, second, or third. If priority is performed second, any exception happens in priority() will results memory leak of newed Widget
*/
```


 

