## Algorithm

LRU cache
https://leetcode.com/problems/lru-cache

To understand it properly is not easy if you never read about lru before. Least recently used seemed straightforward, however, it is harder than I thought, even though read the solution before.
Even when draw, it looks complicated.
eg. capacity = 2, omitting the value, get the following
1>put 1=1: `1`
2>put 2=2: `2 1`
3>get 1: `1 2`
4>put 3=3 `1 3`
5>get 2, not found

The idea

- use linked list
- always put the recently used one(put or get) to the head of the list, so the tail of the list is always the least used
- use map/hashtable to store the location of the node (node pointer) in the linked list

The implementation

- get operation, if found, then move the found node to head
- put operation, if larger than capacity, then remove the last node, and add node to head
- because of the above, the linked list need operation including add(to head) and remove node

At first, I thought about using array to store instead of linked list, then the map stores the index of an item in the array. Since in Go, moving an item in the array can be easily done by re-slicing, which is creating a new memory for the array. But later found it awkward to index between the map and the array, because when it remove a node, it needs to remove by index in the array, but remove by key in the map, and all the map values need to be updated. Moreover, re-slicing is not the target of this problem.

```Go
type LRUCache struct {
    cache map[int]*Node
    head *Node
    tail *Node
    cur int
    capacity int

}

type Node struct{
    key int
    value int
    next *Node
    prev *Node
}

func (this *LRUCache) addNodeToHead(newNode *Node){
    if this.cur == 0 {
        this.head = newNode
        this.tail = newNode
    }else{
        newNode.prev = nil
        newNode.next = this.head
        this.head.prev = newNode
        this.head = newNode
    }
    this.cur++
}

// remove a node which is in the linked list
func (this *LRUCache) removeNode(node *Node){
    if this.cur <= 1{
        this.head = nil
        this.tail = nil
        this.cur = 0
        return
    }

    // cur >=2

    // if node is head
    if node.prev == nil{
        this.head = node.next
        this.head.prev = nil
    }else if node.next == nil{
        this.tail = node.prev
        this.tail.next = nil
    }else{
        node.prev.next = node.next
        node.next.prev = node.prev
    }
    this.cur--
}

func Constructor(capacity int) LRUCache {
    return LRUCache{
        cache:make(map[int]*Node),
        capacity:capacity,
    }
}


func (this *LRUCache) Get(key int) int {
    if node, ok:=this.cache[key]; ok{
        this.removeNode(node)
        this.addNodeToHead(node)
        return node.value
    }
    return -1

}


func (this *LRUCache) Put(key int, value int)  {
    ret:=this.Get(key)
    if ret!=-1{
        this.head.value = value
    }else{
        newNode:=&Node{
            key: key,
            value: value,
        }
        if this.cur < this.capacity{
            this.addNodeToHead(newNode)

        }else{
            // evict
            if this.tail!=nil{
                delete(this.cache, this.tail.key)
            }
            this.removeNode(this.tail)

            this.addNodeToHead(newNode)
        }
        this.cache[key] = newNode
    }


}


/**
 * Your LRUCache object will be instantiated and called as such:
 * obj := Constructor(capacity);
 * param_1 := obj.Get(key);
 * obj.Put(key,value);
 */
```

## Review

I just got a developer job at Facebook. Here’s how I prepped for my interviews.

https://www.freecodecamp.org/news/software-engineering-interviews-744380f4f2af/

Mostly, it is similar to the other interview articles, talks about the importance of algorithm and data structure. In all, it summarizes most of the aspects of an interview, and provides resources that one should study. How you should study the resources, it all depends on your background. I always see people sharing their algorithm solution, but seldom see how to analyze the problem, and find a pattern based on the past knowledge. For each type of algorithm, one can write at least 10 pages discussing all of its variation. In most of the problems, it requires a bit of tricks to convert a new problem into an old one, just like math questions. I guess only when you practice enough, then you build the connection in your brain for pattern recognition.

The points in this article

- Interviewing is a skill, so it can be practiced and learnt.
- Different types of interviews
  - algorithm
  - architecture design: ask the right questions to narrow down the requirements and constraints. eg. modularization, design patterns, scaling up API, decoupling
  - behavioral: be more genuine, admitting your weakness, and show initiatives for improvement
  - culture fit: lookup the company's values beforehand, find matching past experience that you can relate to the interviewer
  - pair programming - interviewer watching you program online: writing scrappy code and think loudly about what should be done in production.
  - finding/patching bugs - only seen once (really? feels like a quiz at school)
  - domain knowledge test - domain knowledge about a specific language's data structure, memory management, why an API is structured in a certain way
  - os questions.
- Preparation
  - write code by hand
  - deep knowledge of data structure: implementing data structures from scratch
  - understanding big O
  - grasp all major sorting algorithms

Note: some company has cooling period as long as 12 months, and once failing, it takes long to try again.

Resources

> Mock Interviews

    - interviewing.io (beta), Free
    - Pramp, Free
    - CareerCup, Paid

> Architecture Design: Intro to Architecture and Systems, YouTube
> Behavioural: Intro to Behavioural Interviews, YouTube

## Tips

FFMpeg basics

- container - Movie file itself is called a container, and the type of container determines where the information in the file goes. Examples of containers are AVI and Quicktime.
- stream - you have a bunch of streams; for example, you usually have an audio stream and a video stream. (A "stream" is just a fancy word for "a succession of data elements made available over time".)
- frame - The data elements in a stream are called frames.
- codec - Each stream is encoded by a different kind of codec. The codec defines how the actual data is COded and DECoded - hence the name CODEC. Examples of codecs are DivX and MP3. packet
- Packets - pieces of data that can contain bits of data that are decoded into raw frames that we can finally manipulate for our application. For our purposes, each packet contains complete frames, or multiple frames in the case of audio.

RTP 协议为了广泛地支持各种多媒体格式（如 H.264, MPEG-4, MJPEG, MPEG），没有在协议中体现出具体的应用配置，而是通过 profile 配置文件及负载类型格式说明文件的形式来提供(sdp file)。

Reference:
https://superuser.com/questions/300897/what-is-a-codec-e-g-divx-and-how-does-it-differ-from-a-file-format-e-g-mp/300997#300997

https://github.com/babosa/Course#course-3

## Share