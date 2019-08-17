# Algorithm

https://leetcode.com/problems/longest-substring-without-repeating-characters/

General idea is, keep two pointers. If from somewhere a character is repeated, and if the rest of the string is part of the longest substring, then the string cannot include the repeated character where it was last found.
Using an example: "tmmat", two pointers i, j
i=0, j=0 m['t'] = 0
i=0, j=1 m['m'] = 1
i=0, j=2 repeated 'm', m['m'] = 2, i = 2. Actually at this point, m['t'] = 0 is not in the search area, should be voided, however, the tricky part is, make sure the pointer always move forward.
i=2, j=3 m['a'] = 3
i=2, j=4 repeated 't', however, since i = 2 > 0, so i does not need to move.

```go
func lengthOfLongestSubstring(s string) int {
    m:=make(map[byte]int)
    i :=0
    maxLen:=0
    for j:=0;j < len(s); j++{
        if inx, ok :=m[s[j]];ok{
            i = max(i, inx+1)
        }
        maxLen =max(maxLen, j-i+1)
        m[s[j]] = j

    }
    return maxLen
}

func max(i, j int) int{
    if i < j {
        return j
    }
    return i
}
```

https://leetcode.com/problems/sort-list/
sort single link list

One solution is merge sort. MergeSort flow is

```
b:=split(a)
a=sort(a)
b=sort(b)
merge(a,b)
```

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func sortList(head *ListNode) *ListNode {
    if head == nil{
        return nil
    }
    if head.Next == nil{
        return head
    }

    s := split(head)
    p := sortList(head)
    q := sortList(s)
    return merge(p, q)

}

func split(h *ListNode) *ListNode {
    slow, fast:=h, h
    for fast!=nil && fast.Next!=nil && fast.Next.Next!=nil{
        fast = fast.Next.Next
        slow = slow.Next
    }
    if slow.Next!=nil{
        n:=slow.Next
        slow.Next = nil
        return n
    }
    return nil
}

func merge(l1 *ListNode, l2 *ListNode) *ListNode{
    sentinel:=&ListNode{}
    h:=sentinel
    for l1!=nil && l2!=nil{
        if l1.Val < l2.Val{
            h.Next = l1
            l1 = l1.Next
        }else{
            h.Next = l2
            l2 = l2.Next
        }
        h = h.Next
    }
    if l1 == nil{
        h.Next = l2
    }else{
        h.Next = l1
    }
    return sentinel.Next
}
```

# Review

[System Design: Distributed Rate Limiting](https://medium.com/@guptaabhinav206/system-design-distributed-rate-limiting-ef7c0650f0e5)
This article talks about some thoughts to avoid cascading failure, dos attack, qos control (high/low priority requests), in a context of distributed environment.

Solutions：

- configure load balancer to stop doing failover, drop the requests. Obviously a bad choice. What's the point of using a cluster if there is no failover.
- control the request rate at client side
- control the request rate at server side
- circuit breaking - use an intermediary to limit the request rate
- queueing the requests, and based on the rest of the node's availability redirect the requests
- pre-scale/auto-scale the system if expecting high traffic
- delay response time/degrade service for low priority requests

Classic rate Limiting Algorithm: https://en.wikipedia.org/wiki/Rate_limiting

1. [Token Bucket](https://en.wikipedia.org/wiki/Token_bucket):
   User need to have enough tokens to get request accepted.

- In cache, it tracks
  `user: last_request_timestamp, token number`
- Each users get tokens refilled at a certain rate or based on last request timestamp.
- Each time a user sends a request, it cost a token. When the token number is 0, request is rejected. The read-and-write to the cache requires atomic operation.

2. Fixed Window counter

- The cache records `user:request_counter` at fixed window interval. For example if limit 5 request per 5 seconds, then there is window of 11:00 - 11:05, 11:05 - 11:10, etc. The request counter is reset at the next window interval.
- Problem: not accurate. eg. if at 11:04 5 request made, and 11:06 5 more request made, then within 11:04 - 11:06, 10 request made, but it is still allowed.

3. Sliding Window log

- Maintain all requests' timestamps from a user. Keep adding timestamp per request, and removed expired timestamp based on the length of throttle requirement.
- Problem: large memory footprint

4. Sliding window counter

- Accumulate the request number within a period of time.
  For example, if we have an hourly rate limit, then record

```
user_1: {
    11:01 - 2 requests
    11:02 - 5 requests
    ...
}
```

All is to say, these are quite straight forward approaches except the bucket one, and sometimes have to balance between performance/memory footprint and accuracy. In that case when you need to do trade off, how to maintain the test cases is a bit tricky.

Relating Concepts:

- Cascading failure - one node's failure causing fail over(by using load balancer) to the rest of the surviving nodes, and causing more overload.
- Load shedder - drop low priority requests to keep the service for high priority ones

Reference:

- https://konghq.com/blog/how-to-design-a-scalable-rate-limiting-algorithm/
- https://medium.com/figma-design/an-alternative-approach-to-rate-limiting-f8a06cf7c94c

# Tips

1. Golang method receiver: [Pointers vs. Values](https://golang.org/doc/effective_go.html#pointers_vs_values)

> The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers. This is because pointer methods can modify the receiver; invoking them on a copy of the value would cause those modifications to be discarded.

2. Passing slice as function parameter.
   Slices are usually modified through an object: create a struct containing a slice, and modify the slice through struct methods.
   However, if slice is passed as a parameter of a function, then it needs to either return the slice, or pass slice's pointer as parameter

Example:

```
func noReturn(a []int, data ...int) {
    a = append(a, data...)
}

func returnS(a []int, data ...int) []int {
    return append(a, data...)
}

var a []int{}
noReturn(a, 1, 2, 3) // a is not changed
a = returnS(a, 1, 2, 3) // append changes will be visible
```

or

```
func foo(s *[]int, data ...int) {
	*s = append(*s, data...)
}

a := []int{1}
foo(&a, 1, 2, 3) // a is changed

```

# Share

## An example to use cmake on lib/dll build in windows

I found building libraries in windows is a bit more complicated in Windows than Linux platform. lib/dll are very confusing at link.

This part illustrate a simplified example to show how to get a basic library build in Windows with cmake. Using cmake can avoid the template in msvc, and with one statement, it exports all the symbols for your library to save effort to define line by line in the file. Even it is targeting on Windows, the configuration can also work on Linux.

### Prerequisite - Install the following

1. msvc (we need a compiler, other compilers should also work)
2. cmake

If built static, a lib file is generated afterwards. When linking the static libraries to the executable, no extra file is needed to run the program. From my understanding, building static is just archiving all obj files into the lib file. At build time, even there are other dependencies, missing `target_link_libraries` can also pass.

If built shared, a lib and a dll file are generated. The dll file is used at run time of the executable. At library build time, specifying the dependent libraries is also required. Without `target_link_libraries` to specify what to link, the build fails.

### code structure

Full code snippet can be found at [here](https://github.com/iyabchen/cross-platform-cmake).

```
|--- app
|   -- addten.cpp
|   -- addten.h
|   -- sum.cpp
|   -- sum.h
|   -- CmakeLists.txt
|--- main
│   -- main.cpp
|   -- CmakeLists.txt
```

The idea is `sum` is a library for summing two numbers together, `addten` uses sum library to add 10 to a number, and `main` invokes `addten` library.
The total number of the programs are less than 20 lines.

```
// sum.h
#pragma once

int sum(int a, int b);

// sum.cpp
int sum(int a, int b)
{
    return a + b;
}
```

```
// addten.h
#pragma once
int addten(int a);

// addten.cpp
#include "sum.h"
#include "addten.h"

int addten(int a)
{
    return sum(a, 10);
}
```

```
// main.cpp
#include "../app/addten.h"
#include <iostream>
int main()
{
    int a = 10;
    std::cout << addten(a) << std::endl;
    return 0;
}

```

Cmake file to build the `sum` and `addten` library.

```
CMAKE_MINIMUM_REQUIRED (VERSION 3.4)
project(app)
if (WIN32)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

add_library(sumlib SHARED sum.cpp sum.h )
add_library(addtenlib SHARED addten.cpp addten.h)
target_link_libraries(addtenlib PRIVATE sumlib)
```

Create a folder called `build` under folder `app`, and run `cmake`.

```
$ mkdir build
$ cd build
$ cmake ..
$ cmake --build .
Microsoft (R) Build Engine version 16.2.37902+b5aaefc9f for .NET Framework
Copyright (C) Microsoft Corporation. All rights reserved.

  Building Custom Rule D:/cross-platform-cmake/app/CMakeLists.txt
  sum.cpp
  Auto build dll exports
     Creating library D:/cross-platform-cmake/app/build/Debug/sum.lib and object D:/cross-platform-cmake/app/build/Debug/sum.exp
  sum.vcxproj -> D:\cross-platform-cmake\app\build\Debug\sum.dll
  Building Custom Rule D:/cross-platform-cmake/app/CMakeLists.txt
  addten.cpp
  Auto build dll exports
     Creating library D:/cross-platform-cmake/app/build/Debug/addten.lib and object D:/cross-platform-cmake/app/build/Debug/addten.exp
  addten.vcxproj -> D:\cross-platform-cmake\app\build\Debug\addten.dll

```

We can see lib and dll files are built under `app/build/Debug`.

Cmake file to build main

```
CMAKE_MINIMUM_REQUIRED (VERSION 3.4)
project(main)
if (WIN32)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

add_executable(main main.cpp "${CMAKE_CURRENT_SOURCE_DIR}/app/sum.h")

find_library(sumlib sumlib "${CMAKE_CURRENT_SOURCE_DIR}/app/Debug")
find_library(addtenlib addtenlib "${CMAKE_CURRENT_SOURCE_DIR}/app/Debug")
target_link_libraries(main ${sumlib} ${addtenlib} )
```

Do the similar to create build folder and run the following commands under `main` folder.

```
$ mkdir build
$ cd build
$ cmake ..
$ cmake --build .
Microsoft (R) Build Engine version 16.2.37902+b5aaefc9f for .NET Framework
Copyright (C) Microsoft Corporation. All rights reserved.

  Checking Build System
  Building Custom Rule D:/cross-platform-cmake/main/CMakeLists.txt
  main.cpp
  main.vcxproj -> D:\cross-platform-cmake\main\build\Debug\main.exe
  Building Custom Rule D:/cross-platform-cmake/main/CMakeLists.txt
```

Copy `sum.dll` and `addten.dll` over to `main/build/Debug`. Then run `main.exe` should work.

```
$ ls
addten.dll*  main.exe*  main.ilk  main.pdb  sum.dll*
$ ./main.exe
20
```

To build static library, simply change `SHARED` to `STATIC` in the cmake file. At main build, both libraries are still required to be linked, but no lib/dll files need to be copied over to run `main.exe`.
There is no way to specify both `SHARED` and `STATIC` together on the same statement, so they have to be specified as libraries with different names.

In windows, build release should use `cmake --build . --config Release`.

### target_link_libraries

No matter shared or static library when adding the library, this command in cmake only adds the dependencies into its link interfaces, prompting the library to know what needs to be linked if this library is used.
Supposed executable A depends on B, B depends on C, when building A, both B and C needs to be listed as link objects, even though when building B it already specified C to link.
As for private, public, interface, it impacts the linked objects' visibility. If A needs to use C's exported function, then C should not be specified as PRIVATE in `target_link_library` statement.

Since A and B both need to link C, and find_library is repeated at both A and B's build. One way to avoid the duplication is to build B and C into one library. In that case, only building A requires to specify what to link.

### Error

1. When executing an exe file, prompt "xxx.dll is needed".

   Probably the lib file you link is not static. From a lib file alone, there is no way to tell whether it is static or dynamic.

   > Not all lib files are static libraries. Some are import libraries, and chances are, that's what you linked with.
   > If your lib file is much smaller than its corresponding dll file, that's a sure sign that it's an import library.

2. Saw "compiler cl is not able to compile a simple test program" when using command line to run cmake, even using the command prompt from msvc.

   After using msvc to create a cmake project, the problem solved, not sure what caused the problem.

Reference:

- https://stackoverflow.com/questions/2240737/static-linking-to-lib-and-still-requesting-dll
- https://gitlab.kitware.com/cmake/cmake/issues/16931
