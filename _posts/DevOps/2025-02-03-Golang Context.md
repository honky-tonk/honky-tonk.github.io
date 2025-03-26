---
layout: post
title: Golang Context
subtitle:
gh-repo: honky-tonk/honky-tonk.github.io 
gh-badge: [star,follow]
tags: [DevOps]
comments: true
---
# What is Golang Context?
In Basically Golang Context is used to Goroutine lifecycle Management and pass small data between multi go routine， let's get some example

example 1:

if a goroutine worker run some operation, but this operation spent time exceed you want, For example a worker run a operation 10 second or more, but 5 second is what you expect, so golang context can do this
```golang
package main

import (
    "fmt"
    "context"
    "time"
    )

func worker(ctx context.Context){
    for{
        select{
            case <- ctx.Done():
                fmt.Println("Worker TimeOut!!!")
                return
        
            default:
                fmt.Println("Worker is Working!!!")
                time.Sleep(500 * time.Millisecond)
        }
    }
    
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 5 * time.Second)
    defer cancel()
    go worker(ctx)
    time.Sleep(6 * time.Second)
}
```
in this example worker will cancel in 5 seconds,we build a ```context``` with timeout and set the time of timeout is 5 sec and pass the context to worker, after 5 sec the context will pass the ```interface{}``` to ```ctx.Done``` channel so we could catch the signal from the channel and exit goroutine , of course you don't want to cancel by timeout, you want to cancel this goroutine by yourself, you can do this 

```golang
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Worker Canceled!!!")
            return
        default:
            fmt.Println("Worker is Working!!!")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background()) 
    go worker(ctx)

    time.Sleep(5 * time.Second)
    cancel()                    // cancel the worker Goroutine
}
```
in this example, ```context.Done()``` channel recive message when  ```cancel()``` function was called by main thread(goroutine),we could pass same context to muliti goroutine and ```cancel()``` them same time 

Also we could use context to pass same message to muilti worker
```golang
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, worker_id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Worker Canceled!!!")
            return
        default:
            fmt.Println("session id: ", "[",ctx.Value("id") , "], ","worker id: ", worker_id,  ", Worker is Working!!!")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    ctx1, cancel := context.WithCancel(context.Background()) /
    ctx2 = context.WithValue(ctx, "id"/*key*/, "CTX-245"/*value*/)
    
    go worker(ctx2, 1)
    go worker(ctx2, 2)

    time.Sleep(5 * time.Second) 
    cancel()                    
    time.Sleep(1 * time.Second) 
}
```
we pass the kv message to 2 worker

# Context tree

the last example we could see a context(```context.Background()```) as root and derive other context(```ctx2``` which is ```context.WithValue()```), this context form a tree, in the nutshell, if root context ```cancel()``` or ```timeout``` the **leaf context** of the tree will cancel too

```golang
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, worker_id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Worker Canceled!!!")
            return
        default:
            fmt.Println("session id: ", "[",ctx.Value("id") , "], ","worker id: ", worker_id,  ", Worker is Working!!!")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    ctx1, cancel1 := context.WithCancel(context.Background()) 
    ctx2 := context.WithValue(ctx1, "id"/*key*/, "CTX-245"/*value*/)
    
    go worker(ctx2, 1)
    go worker(ctx2, 2)

    time.Sleep(5 * time.Second) 
    fmt.Println("root context cancel")
    cancel1()//root context cancel                    
    time.Sleep(1 * time.Second) 
}
```
the root context call ```cancel()``` then each worker exit(the child context pass to worker)




