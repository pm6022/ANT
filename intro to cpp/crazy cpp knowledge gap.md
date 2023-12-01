Create directories for the development Create a directory for the development (same name as the top-level application): 
mkdir appWithClassInLibrary 
cd appWithClassInLibrary 
Create a directory for the class (same name as the class):
mkdir myClass 
cd myClass

Classes should be coded in pairs of files, with a  .H file that contains the class declaration and a .C file that contains the class definition. Put the class declaration in myClass.H:

```cpp
#ifndef myClass_H 
#define myClass_H 
class myClass 
{
private: 
protected: 
public: 
int i_; //Member data (underscore is OpenFOAM convention) 
float j_; 
myClass(); 
~myClass(); 
}; 
#endif

```

This code is a header file for a C++ class called `myClass`. It starts with an include guard to prevent the header file from being included multiple times in the same translation unit.

The include guard is implemented using the preprocessor directives `#ifndef` and `#define`. The first directive `#ifndef` checks if the macro `myClass_H` has not been defined previously. If the macro has not been defined, the `#define` directive defines it, and the subsequent code until the `#endif` directive will be included in the translation unit. If the macro has been defined previously, the subsequent code will be skipped to prevent redefinition errors.

The `class` keyword is used to declare the class `myClass`, which has two member variables declared in the `public` section: an integer `i_` and a floating point number `j_`. The `private` and `protected` sections are currently empty, indicating that all member functions and variables are accessible from outside the class.

The class also declares a default constructor `myClass()` and a destructor `~myClass()`. The constructor and destructor are declared without any arguments, indicating that the class does not require any special initialization or cleanup procedures.

Finally, the include guard is closed with the `#endif` directive to mark the end of the header file.