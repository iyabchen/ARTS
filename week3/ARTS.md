# Algorithm

Common linked list problem : leetcode 206，141，21，19，876.

## 206

https://leetcode.com/problems/reverse-linked-list/

Iteratively

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    var p *ListNode
    q:=head
    for q!=nil{
        n:=q.Next
        q.Next = p
        p = q
        q = n
    }
    return p
}

```

Recursively

```
func reverseList(head *ListNode) *ListNode {
    return reverseListRec(nil, head)
}

func reverseListRec(newHead *ListNode, unprocessed *ListNode) *ListNode{
    if unprocessed == nil{
        return newHead
    }
    n:=unprocessed.Next
    unprocessed.Next = newHead
    newHead = unprocessed
    return reverseListRec(newHead, n)
}
```

## 141

https://leetcode.com/problems/linked-list-cycle/
Using a slow and fast pointer to traverse a linked list.

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func hasCycle(head *ListNode) bool {
    if head == nil{
        return false
    }
    slow:=head
    fast:=head
    for fast.Next!=nil && fast.Next.Next!=nil{
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast{
            return true
        }
    }

    return false
}
```

## 21

https://leetcode.com/problems/merge-two-sorted-lists/

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {

    sentinel:=&ListNode{
        Val: -1,
    }
    p:=sentinel
    for l1!=nil && l2!=nil{
        if l1.Val < l2.Val{
            p.Next = l1
            l1 = l1.Next
        }else{
            p.Next = l2
            l2 = l2.Next
        }
        p = p.Next
    }
    if l1!=nil{
        p.Next = l1
    }else if l2!=nil{
        p.Next = l2
    }
    return sentinel.Next
}
```

## 19

https://leetcode.com/problems/remove-nth-node-from-end-of-list/

Keep two pointers, slow and faster, so that slow one needs n steps of Next operation to reach the faster pointer. When the fast pointer reaches the tail, then the slow pointer delete the next element. Be aware of the nth node definition.

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    sentinel:=&ListNode{
        Val: -1,
        Next: head,
    }
    p,q :=sentinel, sentinel
    for n> 0{
        q = q.Next
        n--
    }
    for q.Next!=nil{
        p = p.Next
        q = q.Next
    }
    p.Next = p.Next.Next
    return sentinel.Next

}
```

## 876

https://leetcode.com/problems/middle-of-the-linked-list/

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func middleNode(head *ListNode) *ListNode {
    slow, fast := head, head
    for fast!=nil && fast.Next != nil{
        slow = slow.Next
        fast = fast.Next.Next
    }
    return slow
}
```

Even understanding the general idea, to use less pointer/space still need some refinement after writing the code, like the original idea requires 3 pointers, but using 2 is also OK and risk the pointer might be nil at the beginning or at the end.

# Review

[Goodbye Docker: Purging is Such Sweet Sorrow](https://zwischenzugs.com/2019/07/27/goodbye-docker-purging-is-such-sweet-sorrow/)

The author stated that a recurring incident causing 100% CPU by the docker daemon became frequent, and the author deleted docker and replace it with podman, skope, and buildah.
Then he introduced what utility each tool brings in corresponding to docker's main features, and the steps to migrate.

The author also mentioned the problem of dockers when he talked about the 3 new tools,

- requires a daemon
- root privileges to a socket
- use vfs storage driver by default -> I think latest docker has used overlay2 by default.

Docker has been used in production for a few if not many years. Personally I do not believe this 100% CPU issue is its pain point, and this article is more like an advertisement/marketing strategy for Redhat. I used Redhat's openshift before, and its user experience for the open source part is not just bad, which looks like a trap to push you buying their support service.

Requiring a daemon to be alive might be an architectural design decision, and I don't see a problem with that. At reboot, the daemon can restart all the contains, without a daemon it is not possible.
Also, I don't understand the technology behind the new tool regarding no root. Docker needs root to mount/unmount filesystems and access the socket. Probably because the contains built by `buildah` are in local user space instead of having system wide access, root can be avoided.

# Tips

Video recording utility

- windows: click on an active window, press win + G
- linux: ctrl + shift + alt + R, default is only 30s length, to change it, install dconf editor and edit

```
/org/gnome/settings-daemon/plugins/media-keys/max-screencast-length - for length
/org/gnome/settings-daemon/plugins/media-keys/screencast - for hotkey
```

# Share

Recently, I was investigating why ffmpeg streaming a video via RTP using a local network got missed packet issue. So far, I have not found the reason yet, but learnt a few things during the search.

The video is streamed from one terminal using ffmpeg.

```shell
# -q 1 is quality, without it, the video is blurry
# -re is using the video's original fps
ffmpeg -re -i /tmp/video.mp4 -sdp_file /tmp/test.sdp -q 1 -f rtp rtp://224.0.1.10:50010
```

The video is played from another terminal, using ffplay or mpv.

```
# need to add udp, rtp to protocol white list
ffplay -protocol_whitelist 'udp,rtp,file,crypto'  /tmp/test.sdp

[sdp @ 0x7fbbd4000b80] max delay reached. need to consume packet
[sdp @ 0x7fbbd4000b80] RTP: missed 4 packets
```

UDP is unreliable protocol, and losing packet is common. However, it happens on a local machine. The video is only playing at less than 5fps rate. The situation seems to get worse when reading with multiple clients.

I got several findings, which might not be related to the cause of the issue, but helped me understand a bit more about ffmpeg, and multicast socket.

1. iperf for testing os stack
   I wasted quite a lot of time just googling the error, to no avail. Not any ffmpeg/ffplay options can help with this.
   Then it came to be to check the IO with `top`, `sar`. Nothing is busy.

```shell
$ sar -n DEV 1
# rxmcst/s is multicast packet per
```

```shell
$ ethtool enp0s3 | grep Speed
```

Even the bandwidth is 100M/s only. However, I calculated the multicast throughput, the util rate is roughly correct.

Then I found `iperf` can do benchmark for multicast.

```shell
# start a terminal for server
# -u udp
iperf -s -B 239.255.1.3 -u -f m -i 5

# start a terminal for client to send data
iperf -c 239.255.1.3 -u -b 990m -f m -i 5 -t 30
```

The loss datagram column should be checked in this case. If any datagram lost at this point, the os stack buffer might not be set large enough.

```
sysctl -a # list all settings
sysctl -w <K>=<V> # modify setting within the current terminal session
# modify /etc/sysctl.conf for a permanent change
```

The following are buffer that can be considered to tune.

```
# read/receive buffer
net.core.rmem_max
net.core.rmem_default
# write/send buffer
net.core.wmem_max
net.core.wmem_default
```

2. ffmpeg bind to INADDR_ANY
   When reading from a multicast address, usually the app will bind to a local address with a random port.

```
$ ss |grep 224
udp   ESTAB   0        0                                192.168.0.22:57301                                        224.0.0.1:9999
udp   ESTAB   0        0                                192.168.0.22:38514                                        224.0.0.1:9999
```

ffplay however, binds to 0.0.0.0:\*, and ss command cannot see that.
https://github.com/FFmpeg/FFmpeg/blob/master/libavformat/udp.c

```
static int udp_join_multicast_group(int sockfd, struct sockaddr *addr,struct sockaddr *local_addr)
{
#ifdef IP_ADD_MEMBERSHIP
    if (addr->sa_family == AF_INET) {
        struct ip_mreq mreq;

        mreq.imr_multiaddr.s_addr = ((struct sockaddr_in *)addr)->sin_addr.s_addr;
        if (local_addr)
            mreq.imr_interface= ((struct sockaddr_in *)local_addr)->sin_addr;
        else
            mreq.imr_interface.s_addr= INADDR_ANY;
        if (setsockopt(sockfd, IPPROTO_IP, IP_ADD_MEMBERSHIP, (const void *)&mreq, sizeof(mreq)) < 0) {
            ff_log_net_error(NULL, AV_LOG_ERROR, "setsockopt(IP_ADD_MEMBERSHIP)");
            return -1;
        }
    }
#endif
```

https://stackoverflow.com/questions/10692956/what-does-it-mean-to-bind-a-multicast-udp-socket

> IP_ADD_MEMBERSHIP - tells your network adapter to listen not only for ethernet frames where the destination MAC address is your own, it also tells the ethernet adapter (NIC) to listen for IP multicast traffic as well for the corresponding multicast ethernet address. Each multicast IP maps to a multicast ethernet address. When you use a socket to send to a specific multicast IP, the destination MAC address on the ethernet frame is set to the corresponding multicast MAC address for the multicast IP. When you join a multicast group, you are configuring the NIC to listen for traffic sent to that same MAC address (in addition to its own).

Using INADDR_ANY will listen to all interfaces.

It is very interesting that, if the stream is on a multicast address, then multiple ffplay can receive the stream by binding to 0.0.0.0, and if the address is the local ip, then ffplay will error out address in use if starting more than one (probably because socket reuse is not enabled).

3. gpu vs cpu for decoding

I noted that using gpu get blocky images more often when compared to cpu. FFMpeg also mentioned using hardware acceleration might not be better than cpu:

> Note that most acceleration methods are intended for playback and will not be faster than software decoding on modern CPUs. Additionally, ffmpeg will usually need to copy the decoded frames from the GPU memory into the system memory, resulting in further performance loss. This option is thus mainly useful for testing.

ffplay does not use hardware acelleration, and there is no option to enable it. Use `mpv` with command `mpv --hwdec=nvdec /tmp/video.sdp` can exploit the gpu.

Ref:

- https://www.ibm.com/support/knowledgecenter/en/SSQPD3_2.6.0/com.ibm.wllm.doc/runningiperfmulti.html
- https://cizixs.com/2018/01/13/linux-udp-packet-drop-debug/
- https://mpv.io/manual/stable/#options-gpu-context
- https://ffmpeg.org/ffmpeg.html
