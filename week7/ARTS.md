# Algorithm

Minimum edit distance https://leetcode.com/problems/edit-distance/

- Initialization: init to length+1. When one string is empty, the other is not, there is still edit distance.
- Core:

  ```
  if word1[i-1] == word2[j-1]{
      arr[i][j] = min(arr[i-1][j-1], arr[i-1][j]+1, arr[i][j-1]+1)
  }else{
      arr[i][j] = min(arr[i-1][j-1], arr[i-1][j], arr[i][j-1])+1
  }
  ```

```
func minDistance(word1 string, word2 string) int {
    l1:=len(word1)
    l2:=len(word2)

    arr:=make([][]int, l1+1)
    for inx:=range arr{
        arr[inx] = make([]int, l2+1)
    }

    for i:=1; i <=l1; i++{
        arr[i][0] = i
    }
    for j:=1; j <=l2;j++{
        arr[0][j] = j
    }

    for i:=1; i <=l1; i++{
        for j:=1; j<=l2; j++{
            if word1[i-1] == word2[j-1]{
                arr[i][j] = min(arr[i-1][j-1], arr[i-1][j]+1, arr[i][j-1]+1)
            }else{
                arr[i][j] = min(arr[i-1][j-1], arr[i-1][j], arr[i][j-1])+1
            }
        }
    }
    return arr[l1][l2]

}

func min(numbers ...int) int{
    m:=numbers[0]
    for _, n:=range numbers{
        if n < m{
            m = n
        }
    }
    return m
}
```

https://leetcode.com/problems/longest-increasing-subsequence/

This problem is not about two strings, but only 1 string.

- The longest length of sequence ending at i maxlis(i) = max(maxlis(j) + 1), where j < i.
- Store the max length for each index, init to 1, since the worse case is 1.
- A better solution https://leetcode.com/problems/longest-increasing-subsequence/discuss/74824/JavaPython-Binary-search-O(nlogn)-time-with-explanation

```
func lengthOfLIS(nums []int) int {
    l:=len(nums)
    arr:=make([]int, l)
    for inx:=range arr{
        arr[inx] = 1
    }

    for i:=0; i<l;i++{
        for j:=0; j<i;j++{
            if nums[j] < nums[i]{
                tmp := arr[j] + 1
                if tmp > arr[i]{
                    arr[i] = tmp
                }
            }
        }
    }

    max:=0
    for _, n:=range arr{
        if n > max{
            max = n
        }
    }
    return max
}
```

# Review

[Why our team cancelled our move to microservices](https://medium.com/@steven.lemon182/why-our-team-cancelled-our-move-to-microservices-8fd87898d952)

The author describe problems in practice regarding why they drop microservice in that scenario.
In my point of view, either microservice or monolithic, the core should still be domain based. Microservice is just a way to organize a project for development, and of course it brings in overhead especially in service communication. Without a clear view for the requirement and business logic, the effort spent on microservices can easily lead to a waste, and you learn something eventually.

The problems they met include

- heavily reliant on a third-party. The product is mostly a passthrough to a third-party app. The third-party team releases, then need to follow up with them.
- couldn’t sufficiently isolate each microservice. The effort left to even more coupling, messages buses everywhere, and a potential big bang of immediately going from one service to ten or more microservices.
- microservices are shared in the team, not able to assign owner ship to any team.
- no supporting platform, like container, kubernetes, gateway.
- the business logic changed too frequently
- tight schedule
- lacking experience

The author also discussed the benefits of microservices to figure out what gain they can get if chasing towards that path

- autonomy - reduce the amount of coordination
- allow to specialize on one service
- easy to scale
- easy to rollback
- easy to release
- use the most appropriate technology (monoliths can be hard to upgrade and stuck on outdated platforms.)
- easy to upgrade
- smaller (easy to understand)

Things to concern:

> logging, monitoring, exception handling, fault tolerance, fallbacks, microservice to microservice communication, message formats, containerization, service discovery, backups, telemetry, alerts, tracing, build pipelines, release pipelines, tooling, sharing infrastructure code, documentation, scaling, timezone support, staged rollouts, API versioning, network latency, health checks, load balancing, CDC testing, fault tolerance, debugging and developing multiple microservices in our local development environment.

> The more we looked into microservices, the more it seemed that it was less about technology and more about structuring teams and the work that came into them.

# Tips

Find Out How Many File Descriptors Are Being Used

```
lsof -p <pid> | wc -l
```

# Share
