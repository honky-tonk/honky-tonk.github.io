---
layout: post
title: Implement Your Own Shared Pointer 
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [C++]
comments: true
---

# Shared Pointer Usage

TODO

# The Principle of Shared Pointer 
TODO

# Implement

```c++
template <typename T>
struct control_block{
    control_block(T* ptr){reference_count = 1; managered_obj = ptr;}
    ~control_block(){}

    T* managered_obj;
    int reference_count;

    void reference_count_add(){ reference_count++; }
    void reference_count_minus(){ reference_count--; }
    int get_reference_count(){ return reference_count; }
};


template <typename T>
struct shared_pointer{
    shared_pointer() = delete;
    shared_pointer(std::nullptr_t){};
    shared_pointer(T* obj = nullptr):
    managered_obj(obj){
        c_b = new control_block<T>(obj);
    }

    shared_pointer(const shared_pointer& shared_ptr):
    c_b(shared_ptr.c_b), managered_obj(shared_ptr.managered_obj){
         c_b->reference_count_add(); 
    }
    shared_pointer operator=(const shared_pointer& ptr);

    shared_pointer(const shared_pointer&& shared_ptr):
    c_b(shared_ptr.c_b), managered_obj(shared_ptr.managered_obj){
        //delete shared_ptr;
    }
    shared_pointer operator=(const shared_pointer&& ptr);

    ~shared_pointer(){ 
        if(c_b->get_reference_count() == 1){

            delete c_b;
            std::cout << "reference count is zero and delete control block" << std::endl;
            //delete managered_obj;
        } else{
            c_b->reference_count_minus();
        }
    }

    control_block<T>* c_b;
    T* managered_obj; 

};

```


