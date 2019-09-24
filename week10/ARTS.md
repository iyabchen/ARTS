# Algorithm

https://leetcode.com/problems/linked-list-cycle-ii/

Circle detection in a linked list. The algorithm is not trivial and contains some math. https://en.wikipedia.org/wiki/Cycle_detection#Brent's_algorithm

The idea of Floyd's algorithm is,

- using slow and faster pointer: faster pointer moves 2 steps each time, while the slower pointer moves only 1 step each time. If there is loop, then fast and slow pointer will meet at a point.
- Two pointer again, one start from head, and the other start from the meeting point. Both of them move 1 step each time, and the next time they meet is the starting of the loop.

To understand this, based on the above two items,

```
  a    b
----x-----y
    |     |
    |_____|
       c
Suppose we have a loop like above
- a = From head to loop starting point x
- b = From loop starting point x to meeting point y
- c = circle length - b

Suppose circle length is r, r = b + c.
Suppose after k steps of slower pointer, they meet at y, by that time, faster pointer already walked n circles in the loop, so
 k = a + b
2k = a + nr + b

Hence, a + b = nr

Then, one pointer start from head, the other start from point y.
Since a = nr - b, so by the time the pointer reach x, the other one should have gone nr - b steps, which is goes forward n circles (reach y again), and goes back b steps, which is x.

```

The code is the following.

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func detectCycle(head *ListNode) *ListNode {
    if head == nil || head.Next == nil{
        return nil
    }
    p, q:=head, head
    for q!=nil && q.Next!=nil{
        p = p.Next
        q = q.Next.Next
        if p == q{
            break
        }
    }
    if p != q{
        return nil
    }

    p = head
    for p!=q{
        p = p.Next
        q = q.Next
    }
    return p
}
```

Interestly, the following question can be transformed to the above one.

https://leetcode.com/problems/find-the-duplicate-number/

```
func findDuplicate(nums []int) int {
    i, j := 0, 0

    l:=len(nums)
    for nums[j] < l {
        i = nums[i]
        j = nums[nums[j]]
        if i == j {
            break
        }
    }
    if i != j {
        return -1
    }

    i = 0
    for i != j{
        i = nums[i]
        j = nums[j]
    }
    return i
}
```

# Review

Already tried event driven architecture before. It helped to focus on domain driven design, but usually the outcome is tightly coupling with one of the event framework. Usually the event frameworks are fine, and flexible enough for different kind of models, but who knows one day where the trend leads.

## [Best Practices for Event-Driven Microservice Architecture](https://www.hackernoon.com/best-practices-for-event-driven-microservice-architecture-e034p21lk)

### What is event driven?

Suppose the message flow is from A to B, service A produce event, and service B subscribes to the event to process further.

### Why

Async, loose coupling, easy scaling, recovery support

### Cons

the most significant drawback and challenge is data and transaction management:

- handle inconsistent data between services
- incompatible versions
- duplicate events
- do not support ACID transactions, instead supporting eventual consistency which can be more difficult to track or debug.

### Event framework

- message processing: Event sends to a specific (and typically single) destination. eg. ActiveMQ, RabbitMQ
- stream processing: Events are not targeted to a certain recipient, but rather are available to all interested components. eg. Kafka

### Consideration

#### Dealing with change

- a good event consumer: means coding for schemas that change.
- a good event producer: being cognizant of how your schema changes impact other services and creating well-designed events that are documented clearly.

#### Don’t use generic events

Events with generic names, or generic events with confusing flags, cause issues.

#### Don't expect event-driven to fix everything

# Tips

In vscode, use regexp to add double quote around a whole line:

search with regexp: (^[0-9A-Za-z_]\*)

replace with: "\$1"

# Share

Even knowing redis can be used for memory cache store, and even persistent data on disk, but never tried it. This part records things that I learnt from https://try.redis.io.

## Basic

### get/set

```
set name "value"
get name // returns value
```

### incr - an atomic operation

```
set x 10
incr x
get x // 11
```

### expiry

`expire` command can set a key's expiry time, in seconds
`ttl` can be used to check how many seconds a key left. -1 means never expire, -2 means already expired.
`set` operation will make the key never expired.

```
> SET resource:lock "Redis Demo 1"
OK
> TTL resource:lock
(integer) -1
> EXPIRE resource:lock 120
(integer) 1
> TTL resource:lock
(integer) 117
> SET resource:lock "Redis Demo 2"
OK
> TTL resource:lock
(integer) -1
```

## data structure

### list data structure

Operations: RPUSH, LPUSH, LLEN, LRANGE, LPOP, and RPOP
(R, L for right, and left, but LLEN, LRANGE?)

LRANGE takes the index of the first element you want to retrieve as its first parameter and **the index of the last element** (not length of the target) you want to retrieve as its second parameter.

```
> LLEN list
(integer) 0
> RPUSH list "a"
(integer) 1
> LPUSH list "b"
(integer) 2
> lrange list 0 -1 // A value of -1 for the second parameter means to retrieve elements until the end of the list.
1) "b"
2) "a
```

### set data structure

Operations: SADD, SREM, SISMEMBER, SMEMBERS and SUNION.

```
> sadd set "a"
(integer) 1
> sadd set "a"
(integer) 0
> smembers set
1) "a"
```

#### sorted set

A sorted set is similar to a regular set, but now each value has an associated score. This score is used to sort the elements in the set.

```
// hackers by year as score
ZADD hackers 1940 "Alan Kay"
ZADD hackers 1906 "Grace Hopper"
ZADD hackers 1953 "Richard Stallman"

// sorted by score from smallest to largest
zrange hackers 0 2
1) "Grace Hopper"
2) "Alan Kay"
3) "Richard Stallman"

```

### hashmap data structure

Operations: HSET, HMSET(multiple set), HGETALL, HGET.
HINCRBY and HDEL can be used for numeric values in the hashmap.
Full list of hashmap commands: http://redis.io/commands#hash

```
> HSET user:1000 name "John Smith"
(integer) 1

> HSET user:1000 email "john.smith@example.com"
(integer) 1

> HGETALL user:1000
1) "name"
2) "John Smith"
3) "email"
4) "john.smith@example.com"

> hget user:1000 email
"john.smith@example.com"
> hget user:1000 other
(nil)


> HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"
OK

```
