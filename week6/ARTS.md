# Algorithm

longest common subsequence https://leetcode.com/problems/longest-common-subsequence/

```
func longestCommonSubsequence(text1 string, text2 string) int {
    l1:=len(text1)
    l2:=len(text2)

    if l1 == 0 || l2 == 0{
        return 0
    }

    arr:=make([][]int, l1+1)
    for i:=range arr{
        arr[i] = make([]int, l2+1)
    }


    for i:=1; i<= l1; i++{
        for j:=1; j<=l2; j++{
            if text1[i-1] == text2[j-1]{
                arr[i][j] = max(arr[i-1][j-1]+1,arr[i][j-1], arr[i-1][j])
            }else{
                arr[i][j] = max(arr[i-1][j-1],arr[i][j-1], arr[i-1][j])
            }
        }
    }
    return arr[l1][l2]
}

func max(numbers ...int) int{
    m:=numbers[0]
    for _, n:=range numbers{
        if n > m{
            m = n
        }
    }
    return m
}
```

vs

longest common substring
https://leetcode.com/problems/maximum-length-of-repeated-subarray

- the array is not recording the longest common substring length so far, but just max common substring ending at i, j
- use another variable to record the longest length

```
func findLength(A []int, B []int) int {

    l1:=len(A)
    l2:=len(B)

    ret:=0

    arr:=make([][]int, l1+1)
    for inx:=range arr{
        arr[inx] = make([]int, l2+1)
    }

    for i:=1; i <=l1; i++{
        for j:=1; j<=l2; j++{
            if A[i-1] == B[j-1]{
                arr[i][j] = arr[i-1][j-1] + 1
                ret = max(ret, arr[i][j])
            }
        }
    }
    return ret
}

func max(numbers ...int) int{
    m:=numbers[0]
    for _, n:=range numbers{
        if n > m{
            m = n
        }
    }
    return m
}
```

# Review

[Diving deep into the golang channels](https://codeburst.io/diving-deep-into-the-golang-channels-549fd4ed21a8)

This article shows how the source code work for channels, explaining the data structure, and several go channel use case from source code.

## channel data struct

```
type hchan struct {


	buf      unsafe.Pointer // points to an array of dataqsiz elements
	dataqsiz uint           // size of the circular queue, buffer size
	qcount   uint           // total data in the queue
    // The above are only for buffered channel

	elemsize uint16 // the size of a channel corresponding to a single element
	closed   uint32 // 0 is open, 1 is closed
	elemtype *_type // element type

	sendx    uint   // send index,
	recvx    uint   // receive index
    // The above 2 are state field of a ring buffer, which indicates the current index of buffer — backing array from where it can send data and receive data.

	recvq    waitq  // list of recv waiters/go routines
	sendq    waitq  // list of send waiters/go routines

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}

type waitq struct {
	first *sudog
	last  *sudog
}
```

sudog represents a g in a wait list, such as for sending/receiving on a channel. Double linked list.

```
// sudog is necessary because the g ↔ synchronization object relation
// is many-to-many. A g can be on many wait lists, so there may be
// many sudogs for one g; and many gs may be waiting on the same
// synchronization object, so there may be many sudogs for one object.
//
// sudogs are allocated from a special pool. Use acquireSudog and
// releaseSudog to allocate and free them.
type sudog struct {
	// The following fields are protected by the hchan.lock of the
	// channel this sudog is blocking on. shrinkstack depends on
	// this for sudogs involved in channel ops.

	g *g

	// isSelect indicates g is participating in a select, so
	// g.selectDone must be CAS'd to win the wake-up race.
	isSelect bool
	next     *sudog
	prev     *sudog
	elem     unsafe.Pointer // data element (may point to stack)

	// The following fields are never accessed concurrently.
	// For channels, waitlink is only accessed by g.
	// For semaphores, all fields (including the ones above)
	// are only accessed when holding a semaRoot lock.

	acquiretime int64
	releasetime int64
	ticket      uint32
	parent      *sudog // semaRoot binary tree
	waitlink    *sudog // g.waiting list or semaRoot
	waittail    *sudog // semaRoot
	c           *hchan // channel
}
```

Send operation Summary

1. lock the entire channel structure.
2. determines writes. Try recvq to take a waiting goroutine from the wait queue, then hand the element to be written directly to the goroutine.
3. If recvq is Empty, Determine whether the buffer is available. If available, **copy** the data from current goroutine to the buffer.
4. If the buffer is full then the element to be written is saved to the sudog structure and enqueued at sendq and suspended from runtime.
   (Receive operation is similar)

## Select

Each case in select is represented by scase.

```
type scase struct {
    c           *hchan
    elem        unsafe.Pointer
    kind        uint16 // including  caseNil, caseRecv, caseSend, caseDefault
    pc          uintptr
    releasetime int64
}
```

A lock order is calculated by each scase's hchan address, random is used.
Loop from the lock order, if any case does not block, then the select statement can return.
If no channel currently responds and there is no default statement, current g must currently hang on the corresponding wait queue for all channels.

Reference

- https://draveness.me/golang/keyword/golang-select.html

# Tips

If elasticsearch content has a field @timestamp, then search index by time order can be done like

```
curl localhost:9200/<index_name>/_search?pretty -d '{
 "query": {
   "match_all": {}
 },
 "size": 1,
 "sort": [
   {
     "@timestamp": {
       "order": "desc"
     }
   }
 ]
}'
```

# Share

#### Kafka SSL and SASL

Kafka SSL is used for encryption in transportation layer, and SASL is used for authentication. Recently, I found SSL is also used for authentication.

- When Kafka listener is `SSL`, without `SASL`, the broker requires client to provide a client cert and client key, and the server authenticate with its truststore.

- When Kafka listener is `SSL_SASL`, then the client authentication is done through SASL. The client key pair is not required, but SASL authentication credential is needed.

When SSL is enabled, at least 1-way authentication is enforced, which is the client authenticates the server certificates. When `ssl.client.auth` set to `required`, then 2-way authentication is enforced. It can use SSL, which is the broker authenticate the client certificate, or SASL etc.

Reference

- https://docs.confluent.io/current/kafka/authentication_ssl.html
- Kafka keystore and truststore
  https://github.com/confluentinc/confluent-platform-security-tools/blob/master/single-trust-store-diagram.pdf
- https://robertheaton.com/2014/03/27/how-does-https-actually-work/
