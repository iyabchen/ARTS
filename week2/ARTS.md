## Algorithm

Kth Largest Element in an Array
https://leetcode.com/problems/kth-largest-element-in-an-array/

Solution

- use quick sort
- get pivot. if first half is larger than k, then k-th element is in the first half.

The partition algorithm is a bit counterintuitive for me. From instinct, we traverse every item to make sure it matches the condition, however, in that algorithm, it traverse the array to find the item that does not match the condition. Similar problem can be seen in https://leetcode.com/problems/move-zeroes.

Quoting the explanation from [here](https://time.geekbang.org/column/article/41913). For partition func running on array A[p..r], A[p..j-1] is the processed area, and A[j..r] is the unprocessed area. Every time a number is taken out from the unprocessed area (index i), and compared with the pivot. If it match the condition, then it is added to the end of the processed area (index j).

```
func findKthLargest(nums []int, k int) int {
    lo, hi:=0, len(nums)-1
    for lo < hi{
        p :=partition(nums, lo, hi)
        fmt.Println(p)
        if k-1 == p{
            return nums[p]
        }else if k - 1 < p{
            hi=p-1
        }else{
            lo= p+1
        }
    }
    return nums[lo]

}

func partition(a []int, lo, hi  int) int{
    pivot:=a[hi]
    j:=lo
    for i:=lo;i<hi;i++{
        if a[i] > pivot {
            a[j], a[i] = a[i], a[j]
            j++
        }
    }
    a[j], a[hi] = pivot, a[j]
    return j

}
```

## Review

[The Rise and Fall of Object Oriented Programming](https://medium.com/machine-words/the-rise-and-fall-of-object-oriented-programming-d67078f970e2)

My view for this article

- I feel most of the points, even the functional programming part!
- An good example to learn English
  The author mentioned pain points in OOP, no solution is described yet. Golang, in most of the way, does abandon some OOP features to achieve more clarity.

![playtus](playtus.jpg)

OOP Problems

- An object(playtus) does not belong to any existing category, and creating a new one "can have significant costs in terms of effort and program complexity".
- Inheritance. Too many layers and methods causing unnecessary complexity. The author prefer composition > inheritance.
- Objects and methods aren't real. They are "more a reflection of our human psychology than physical reality".
- Visibility brings in complexity for writing test cases.
- Relational database. Data are not like objects, but "I tend to think of relational streams of data as more like a fluid, where you divide, transform, and combine data using algebraic operations."
- Functional programming. No need to stick to pure functional programming, use when applicable and appropriate. "the pure approach tends to take certain relatively straightforward programming exercises and turn them into puzzles"

## Tips

Linux utilities for checking file symbols: `readelf`, `nm`, `ldd`, `c++filt`, `strings`. It comes very handy when an error pops up saying libxxx not found.

```
# ldd can check executable depends on which libraries
/bin$ ldd ls
	linux-vdso.so.1 (0x00007ffe803fa000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007fb991b4b000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fb99175a000)
	libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007fb9914e8000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fb9912e4000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fb991f95000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fb9910c5000)
```

```
# can read libraries(*.so, *.a), and executables
$ readelf -s libau.so

Symbol table '.dynsym' contains 50 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getenv@GLIBC_2.2.5 (2)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND free@GLIBC_2.2.5 (2)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strcasecmp@GLIBC_2.2.5 (2)
...
```

```
# list symbols, similar to readelf
$ nm -D libau.so
                 U calloc
                 U close
0000000000001a58 T closedir
                 w __cxa_finalize
                 U dirfd
                 U dlerror
...
```

Quote from https://stackoverflow.com/questions/9961473/nm-vs-readelf-s
There are regular symbol table, and dynamic symbol table. Regular one is mostly for debug purpose, and strip can remove it.

```
# c++filt can demangle names

$ nm libboost_atomic.a

lockpool.o:
0000000000000000 b _ZN5boost7atomics6detail12_GLOBAL__N_1L11g_lock_poolE
0000000000000000 T _ZN5boost7atomics6detail8lockpool11scoped_lockC1EPVKv
0000000000000000 T _ZN5boost7atomics6detail8lockpool11scoped_lockC2EPVKv
0000000000000060 T _ZN5boost7atomics6detail8lockpool11scoped_lockD1Ev
0000000000000060 T _ZN5boost7atomics6detail8lockpool11scoped_lockD2Ev
0000000000000080 T _ZN5boost7atomics6detail8lockpool12signal_fenceEv
0000000000000070 T _ZN5boost7atomics6detail8lockpool12thread_fenceEv

$ nm libboost_atomic.a|c++filt

lockpool.o:
0000000000000000 b boost::atomics::detail::(anonymous namespace)::g_lock_pool
0000000000000000 T boost::atomics::detail::lockpool::scoped_lock::scoped_lock(void const volatile*)
0000000000000000 T boost::atomics::detail::lockpool::scoped_lock::scoped_lock(void const volatile*)
0000000000000060 T boost::atomics::detail::lockpool::scoped_lock::~scoped_lock()
0000000000000060 T boost::atomics::detail::lockpool::scoped_lock::~scoped_lock()
0000000000000080 T boost::atomics::detail::lockpool::signal_fence()
0000000000000070 T boost::atomics::detail::lockpool::thread_fence()

```

```
# list printable strings

$ strings /bin/ls|head
/lib64/ld-linux-x86-64.so.2
libselinux.so.1
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable

```

## Share
