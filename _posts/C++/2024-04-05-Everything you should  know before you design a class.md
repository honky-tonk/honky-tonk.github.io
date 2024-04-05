---
layout: post
title: Everything you should  know before you design a class
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [C++]
comments: true
---
# Preface
When we design a class, we really should take a second thought, because most of problem are caused by improper class design in particular a pointer inlcuded in class, so before we design we should considering the Copy Construct Move Construct,

# Copy Construct
First we take a look below code.
```cpp
#include <iostream>

class A{
public:
    A(){ptr1 = new int(5);}
    ~A(){delete ptr1;}
    A(const A& a) = default;
    int* getPtr() const {return ptr1;}
private:
    int * ptr1;
};

int main(){
    A a1;
    A a2(a1);
    std::cout << "Get a1 ptr1 address: " << a1.getPtr() << std::endl;
    std::cout << "Get a2 ptr1 address: " << a2.getPtr() << std::endl;
}
``` 
In this code when ```a1 = a2``` exec, a1 call **default** copy construct, in default copy construct, the pointer member of class will NOT deep copy(copy the memory which pointer point to) but shallow copy(just copy pointer value), so in the code above object a1 and a2 has pointer member and the two pointer member point to same memory address, this unsafe, and when we destruct object, the two class will free the same memory address TWICE!!!
So we need rewrite the copy contruct instead of default  construct, after edit the correct code show below
```c++
#include <iostream>

class A{
public:
    A(){ptr1 = new int(5);}
    ~A(){delete ptr1;}
    A(const A& a){ptr1 = new int(*a.getPtr());}
    int* getPtr() const {return ptr1;}
private:
    int * ptr1;
};

int main(){
    A a1;
    A a2(a1);
    std::cout << "Get a1 ptr1 address: " << a1.getPtr() << std::endl;
    std::cout << "Get a2 ptr1 address: " << a2.getPtr() << std::endl;
}
```
# Copy Assignment Operator
Just like copy construct The copy constructor can also easily lead to two pointers pointing to the same block of memory. the different between copy construct and  copy constructor is the way call them.
```c++
A a2
A a1(a2); //call copy constructor
A a1, a2;
a1 = a2; //call copy assignment
A a1 = a2 // call copy constructor
```
so the incorrect code as follow 
```c++
#include <iostream>

class A{
public:
    A(){ptr1 = new int(5);}
    ~A(){delete ptr1;}
    //A(const A& a){ptr1 = new int(*a.getPtr());}
    A& operator=(const A& )=default;
    int* getPtr() const {return ptr1;}
private:
    int * ptr1;
};

int main(){
    A a1;
    A a2;
    a2 = a1;
    std::cout << "Get a1 ptr1 address: " << a1.getPtr() << std::endl;
    std::cout << "Get a2 ptr1 address: " << a2.getPtr() << std::endl;
}
```
after modified the correct code as follow
```c++
#include <iostream>

class A{
public:
    A(){ptr1 = new int(5);}
    ~A(){delete ptr1;}
    //A(const A& a){ptr1 = new int(*a.getPtr());}
    A& operator=(const A& a){
        if(ptr1){ //avoid memory leak!!!
            delete ptr1;
        }
        ptr1 = new int(*a.getPtr()); 
        return *this;
        }
    int* getPtr() const {return ptr1;}
private:
    int * ptr1;
};

int main(){
    A a1;
    A a2;
    a2 = a1;
    std::cout << "Get a1 ptr1 address: " << a1.getPtr() << std::endl;
    std::cout << "Get a2 ptr1 address: " << a2.getPtr() << std::endl;
}
```
we should notice the a2 has their own ptr1, so before deep copy we should free origin memory of ptr1 point to.
# Move Construct
before we get start we should know the key concept of before C++ 11 ```lvalue```, ```rvalue```,and concept after C++11  ```xvalue```, ```glvalue```,```prvalue```.

**lvalue**: so-called, historically, because lvalues could appear on the left-hand side of an assignment expression,lvalue do have memory address

**rvalue**: so-called, historically, because rvalues could appear on the right-hand side of an assignment expression, rvalue does not have memory address,rvalue store in register or memory 

After C++11 standard move semantics and rvalue references xvalue is introduced

**xvalue**: (an “eXpiring” value) also refers to an object, usually near the end of its lifetime (so that its resources may be moved, for example). An xvalue is the result of certain kinds of expressions involving rvalue references.xvalue is a type of lvalue, and xvalue is destroy or expiring quickly

**glvalue**: is an lvalue or an xvalue.

**prvalue**: (“pure” rvalue) is an rvalue that is not an xvalue.

First we move a string to other string
```c++
#include <string>
#include <iostream>

int main(){
    std::string s1 = "hello world";
    std::string s2;

    std::cout << "s1 is: " << s1 << std::endl;
    std::cout << "s2 is: " << s2 << std::endl;   

    s2 = std::move(s1);

    std::cout << "s1 is: " << s1 << std::endl;
    std::cout << "s2 is: " << s2 << std::endl;  
}
```

output:
```
s1 is: hello world
s2 is: 
s1 is: 
s2 is: hello world
```
the string of s1 is moved to string s2 and stri
ng of s1 is gone!
so why? why s1's string is gone and move to s2? actully it's kind of easy, s1 like a pointer, point to string "hello world", in ```std::move```, the pointer pass to s2, and clear s1, so no copy happen in detail, just pointer pass, it;s pretty high performence, **in other word```s1 = std::move(s1)```, make s1 a lvalue convert to xvalue and return string "hello word" 's rvalue reference to s2, the end xvalue(s1) is expired.**

now this another example

```c++
#include <string>
#include <iostream>

int main(){
    int i1 = 8;
    int i2 = 0;

    std::cout << "i1 is: " << i1 << std::endl;
    std::cout << "i2 is: " << i2 << std::endl;   

    i2 = std::move(i1);

    std::cout << "i1 is: " << i1 << std::endl;
    std::cout << "i2 is: " << i2 << std::endl;  
}
```
output
```
i1 is: 8
i2 is: 0
i1 is: 8
i2 is: 8
```
```std::move``` did work for int type!,why? because int type does have move constructor, bu std::string have, so the std::string's move constructor make string's pointer move to peer(convert paramater to xvalue and return origin string's rvalue reference), and erase origin one(xvalue expired), for int type, ```std::move``` is low profermence, because int need copy all memory block to peer to finish ```std::move```, so this is why code below output like that

now we should known why we should design move construct properly, because if we did not write our own move construct their will copy in ```std::move```

```c++
#include <string>
#include <iostream>

class A{
public:
    A(){ptr = new int(8);}
    A(const A& ){std::cout << "copy construct" << std::endl;}
    int* getPtr() const {return ptr;}
private:
    int* ptr;
};

int main(){
    A a1;
    A a2 = std::move(a1);
}
```
output
```
copy construct
a1's ptr is: 0x21d22b0
a2's ptr is: 0
```
see in std::move a2 call copy construct when we not have move construct, and did not copy anything, if we fill the copy construct code like we list before, they copy all the memory of ptr it's not efficiency, so how to design the move construct? like follow code.
```c++
#include <string>
#include <iostream>

class A{
public:
    A(){ptr = new int(8);}
    ~A(){
        if(ptr != nullptr){
            delete ptr;
        }
    }
    A(const A& a){
        std::cout << "copy construct" << std::endl;
        ptr = new int(*a.getPtr());
    }
    A( A&& a){
        std::cout << "move construct " << std::endl;
        ptr = a.getPtr();
        a.setPtr(nullptr); 
    }

    int* getPtr() const {return ptr;}
    void setPtr(int* p){ptr = p;}
private:
    int* ptr;
};

int main(){
    A a1;
    std::cout << "a1's ptr is: " << a1.getPtr() << std::endl; 
    A a2 = std::move(a1);
    std::cout << "a1's ptr is: " << a1.getPtr() << std::endl;
    std::cout << "a2's ptr is: " << a2.getPtr() << std::endl;
}
```
so that is! after move construct a1 own the memory which a1's ptr point to, in ```std::move(a1)```a1's ptr pass to a2 ptr, and a1's ptr is expire(set to nullptr), before move a2 ptr have the memory a1's used to have.