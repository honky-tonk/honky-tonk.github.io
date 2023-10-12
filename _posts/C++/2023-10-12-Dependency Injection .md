---
layout: post
title: C++ dependency injection
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star, fork, follow]
tags: [C++]
comments: true
---

In C++ class design, we often face various challenges related to project scalability and maintainability. As the project requirements expand, classes tend to become more complex. For example, Class A may depend on Class B. However, if there is a need to modify Class B due to a requirement change, it can also necessitate a redesign of Class A.

This kind of interdependence between classes can lead to difficulties in maintaining the codebase. Changes made in one class may require modifications in other classes that depend on it. This issue becomes particularly evident in larger projects where numerous classes have interdependencies.

for example, i got serval classes , in this classes  class ```Garage``` depend on class ```Car```
```c++
#include <iostream>
#include <string>

class Vehicle{
public:
    Vehicle(){std::cout << "construct class Vehicle " << std::endl;};
    ~Vehicle(){std::cout << "destruct class Vehicle " << std::endl;};
    virtual void Print_Type() = 0;
};

class Car: public Vehicle{
public:
    Car(){std::cout << "construct class Car " << std::endl;};
    ~Car(){std::cout << "destruct class Car " << std::endl;};
    virtual void Print_Type(){ std::cout << "Car type" << std::endl; };
};

class Bike: public Vehicle{
public:
    Bike(){std::cout << "construct class Bike " << std::endl;};
    ~Bike(){std::cout << "destruct class Bike " << std::endl;};
    virtual void Print_Type(){ std::cout << "Bike type" << std::endl; };
};


class Garage{
public:
    Garage(){std::cout << "construct class Garage " << std::endl;};
    ~Garage(){std::cout << "destruct class Garage " << std::endl;};
    void get_car_type(){ c.Print_Type(); };

private:
    Car c;
};

int main(){
    Garage g;
    g.get_car_type();

}
```
Everything may seem fine, but when a new requirement arises that necessitates splitting the class ```Car``` into ```Electric_Car``` and ```Petrol_Car```, we need to make some adjustments. In the existing code, we would need to create two new classes, ```Electric_Car``` and ```Petrol_Car```, both inheriting from the  class Car. Additionally, we would also have to redesign the Garage class to accommodate these changes. This can be a tricky situation to handle.

in this scenario, all we need is **break** the dependency

how we break the dependency? **dependency injection**

we no need ```Car c``` in class ```Garage```, instead ```Car* c```, and we pass the pointer of ```Electric_Car``` or  ```Petrol_Car``` to ```Car* c```like following code
```c++
#include <iostream>
#include <string>

class Vehicle{
public:
    Vehicle(){std::cout << "construct class Vehicle " << std::endl;};
    ~Vehicle(){std::cout << "destruct class Vehicle " << std::endl;};
    virtual void Print_Type() = 0;
};

class Car: public Vehicle{
public:
    Car(){std::cout << "construct class Car " << std::endl;};
    ~Car(){std::cout << "destruct class Car " << std::endl;};
    virtual void Print_Type(){ std::cout << "Car type" << std::endl; };
};

class Electric_Car: public Car{
public:
    Electric_Car(){std::cout << "construct class Electric_Car " << std::endl;};
    ~Electric_Car(){std::cout << "destruct class Electric_Car " << std::endl;};
    virtual void Print_Type(){ std::cout << "Electric_Car type" << std::endl; };
};

class Petrol_Car: public Car{
public:
    Petrol_Car(){std::cout << "construct class Petrol_Car " << std::endl;};
    ~Petrol_Car(){std::cout << "destruct class Petrol_Car " << std::endl;};
    virtual void Print_Type(){ std::cout << "Petrol_Car type" << std::endl; };
};

class Bike: public Vehicle{
public:
    Bike(){std::cout << "construct class Bike " << std::endl;};
    ~Bike(){std::cout << "destruct class Bike " << std::endl;};
    virtual void Print_Type(){ std::cout << "Bike type" << std::endl; };
};


class Garage{
public:
    Garage(Car*&& car){
        c = car;
        std::cout << "construct class Garage " << std::endl;
        };
    ~Garage(){std::cout << "destruct class Garage " << std::endl;};
    void get_car_type(){ c->Print_Type(); };

private:
    Car* c; //break dependency
};

int main(){
    Garage g(std::move(new Electric_Car));
    g.get_car_type();

}
```

One thing we need to figure out is what happens when a derived class object pointer points to a parent class object pointer. In my personal view, when a derived class object pointer points to a parent class object pointer and we use the parent class object pointer, it behaves as if we are using the parent class object pointer. However, the virtual field is replaced with the derived class object which overrides it. In the code example above, we pass a ```Electric_Car``` object to its parent class ```Car```. When we use the class ```Car``` object pointer, it behaves like a ```Car``` object (because it is a Car object!). But the pointer points to the class ```Electric_Car```, so this means we use the privileges of the class ```Car``` to access fields of the ```Electric_Car object```. In the memory layout of the Electric_Car object, there is a parent class ```Car```'s vptr (virtual function pointer), which is accessible for ```Car```. However, the virtual function that the vptr points to is replaced with the one from the class ```Electric_Car```.


