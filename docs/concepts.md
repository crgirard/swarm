## The `send` command

Understanding the 'send' command is critical for effective programming in Swarm. **Agents** don't communicate through traditional function calls, such as the following (Python):

```
def doubleIt(n):
    return n*2

def tripleIt(n):
    return n*3

def f(n):
    return tripleIt(doubleIt(n))

print(f(1))
```
Here, `f` is called, and execution of everything else stops. Then `f` calls `doubleIt`, and `f` stops to wait for the results returned by `doubleIt`. Once `f` resumes, it then calls `tripleIt` again, and again waits for results. Finally, it returns the result back to the `print` function. **Swarm** handles this rather differently:

```
define main:
    init:
        1 -> doubleIt
        
define doubleIt(n):
        n*2 -> tripleIt
        
define tripleIt(n):
        n*3 -> print
```
The send command (`->`) sends the data on its left to the **agent** on its right. Mechanically, this can be modelled as the following: **agents** have a data queue, and they repeatedly pop items from the front of the queue and execute themselves on that data, for as long as there is data in the queue. The send command appends a piece of data to the back of that queue. This way, it is possible for these components to communicate in a non-blocking fashion.




## Agents

Basic program structure is defining a set of **agents**:

```
define average:
    init:
        total,n = 0,0

    run(e):
        total,n = total+e,n+1
        total/n -> print
```

**Agents** have two specially named parts:

- `init` is executed once initially
- `run` is executed repeatedly, every time data is sent to the function/agent

```
define total:
    init:
        t = 0

    run(e):
        t = t + e
        t -> print
```        

```
define fibonacci:
   init:
       (0,0) -> fibonacci
   
   run(a,b):
       a -> print
       (b,a+b) -> fibonacci
```

Any number of other inputs can be defined, sent to via the `.` command, for example `agent.input`. Sending data to the function name itself, ie `-> agent` is equivalent to sending to the `.run` subagent, ie `-> agent.run`

```

define example:
    init:
        # do stuff
        
    run(n):
        n + ' received by example.run' -> print
        
    a(n):
        n + ' received by example.a' -> print

    b(n):
        n + ' received by example.b' -> print

define test:
    init:
        5 -> example
        4 -> example.run
        3 -> example.a
        2 -> example.b
```

```
5 received by example.run
4 received by example.run
3 received by example.a
2 received by example.b
```

Both `init` and `run` are optional. The following is completely valid:

```
define example:
    alternateInput(n):
        # do things
```

In the above example, the command `42 -> example` would be invalid (because `example.run` hasn't been defined), but `42 -> example.alternateInput` would work.


When `run` is the only section defined, it can be abridged. The following two definitions are equivalent:
```
define example:
    run(n):
        n -> other
```
```
define example(n):
        n -> other
```

You can imagine each **agent** as a process, and each **subagent** as a thread within that process. the `init` command, when used within an **agent**'s definition, will initialize variables that are shared among all **subagents**


## Scope

```
define driver:
    init:
        for n in [1:3]:
            n -> agent

define agent:
    init:
        self.e = 4
    
    run(q):
        self.e -> 4
```
```
4
4
4
```
```
define driver:
    init:
        for n in [1:3]:
            n -> agent

define agent:
    init:
        e = 4
    
    run(q):
        e -> 4
```
```
ERROR: 'e' is not defined
```
```
define driver:
    init:
        for n in [1:3]:
            n -> agent

define agent:
    init:
        self.e = 4
    
    run(q):
        self.e = q
        self.e -> 4
```
```
1
2
3
```
```
define agent:
    init:
        e = 10
        e -> print
        for e in [0:3]:
            e -> print
        e -> print
```
```
10
0
1
2
3
3
```
```
define agent:
    init:
        e = 10
        e -> print
        for e in [0:3]:
            e *= 2
            e -> print
        e -> print
```
```
10
0
2
4
6
6
```





















