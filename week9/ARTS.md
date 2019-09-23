# Algorithm

### Backtracking

This topic is harder than I thought, because recursion is involved, and not as easy analyzing as a tree. The time complexity is high, but even then, how to write the algorithm is not easy to figure out.

[Here](<https://leetcode.com/problems/subsets/discuss/27281/A-general-approach-to-backtracking-questions-in-Java-(Subsets-Permutations-Combination-Sum-Palindrome-Partitioning)>) mentioned a general approach for backtracking problems.

When it comes to golang, note that every time appending/changing a slice, the underlying array will be change. Every time do a copy, need to do a deep copy for the slice, if the slice is going to be changed in the rest of the backtracking.

For golang, the template looks like

```
func solution(nums []int) [][]int {
    // optionally, sort the array, depends on the problem
    sort.Slice(nums, func(i, j int)bool{
        return nums[i]<nums[j]
    })

    h:=&helper{
        result : make([][]int, 0),
    }
    // recursively run bt
    h.bt(nums, []int{}, 0)
    return h.result
}

// helper to store a shared result between each recursion
type helper struct{
    result [][]int
}

func (h *helper) bt(nums []int, temp []int, start int){
    // append the matched ones to final result
    // using a copy to avoid array content change
    if matched {
        t:=make([]int, len(temp))
        copy(t, temp)
        h.result = append(h.result, t)
    }

    // how to loop and handle the middle result depends on the problem
    for i:=start; i < len(nums); i++    {
        // middle result
        temp = append(temp, nums[i])
        // recursively handle next sub-problem
        h.bt(nums, temp, i+1)
        // restore the middle result to previous state
        temp = temp[:len(temp)-1]
    }
}
```

Take permutation as an example.
https://leetcode.com/problems/permutations/

It can be handled like this:

    Choose a first number, put it into middle result, then handle the rest n-1 array.

    eg. input is [1,2,3].
    f(n) = f([1,2,3]) = {f(n-1), 1} + {f(n-1), 2} + {f(n-1), 3}

The way to loop through the problem is the tricky part.

One way to do that is, loop from 1 ~ n, always swap the current number with the last number, so the next sub problem handles 1 ~ n-1.

Or, loop from 1 ~ n, take out the current number, and create an array to hold the rest of the number, the new array go into the next sub-problem.

```
// solution 1, swap nums[i] with nums[k-1]
// then in next subsequent recursion, it handles 1~k, 1~k-1, ..., k initially set to n.

func (h *helper) bt(nums []int, k int, temp []int){
    if k == 0{
        t:=make([]int, len(temp))
        copy(t, temp)
        h.result = append(h.result, t)
        return
    }
    for i:=0 ; i < k; i++{
        temp = append(temp, nums[i])
        nums[i], nums[k-1] = nums[k-1], nums[i]
        h.bt(nums, k-1, temp)
        nums[i], nums[k-1] = nums[k-1], nums[i]
        temp = temp[:len(temp)-1]
    }
}

// solution 2, make new array, wasted extra space, but easier for me to understand
func (h *helper) bt(nums []int, temp []int){
    if len(nums) == 0{
        t:=make([]int, len(temp))
        copy(t, temp)
        h.result = append(h.result, t)
        return
    }
    for i:=0 ; i < len(nums); i++{
        temp = append(temp, nums[i])
        newArr:=[]int{}
        if i == 0{
            newArr = append(newArr, nums[i+1:]...)
        }else{
            newArr = append(newArr, nums[:i]...)
            newArr = append(newArr, nums[i+1:]...)
        }

        h.bt(newArr, temp)

        temp = temp[:len(temp)-1]
    }
}

```

Time complexity is O(n!).

# Review

[An Introduction to OAuth 2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)

App A wants to use B's service. Instead of surrendering B's user account and password to app A, and oauth authorization can be used. It "works by delegating user authentication to the service that hosts the user account, and authorizing third-party applications to access the user account."

Usually the process of oauth is like, user is visiting app A, and A needs B's resource, so A redirects user to B's website, login there, and authorize A's access to B's resource, then if granted, redirect back to A's webpage.

Many websites in Canada lack this kind of service. Only federal government use similar idea to login from the banks' website. Even paypal asks user to surrender banking account's user name and password, which make most of the financial related services in Canada look unsafe.

This article introduces how Oauth works, and the major terminologies in OAuth. For a more detailed introduction with RFC quotes, check http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html.

OAuth defines four roles as shown in the flow picture.

- Resource Owner
- Client
- Resource Server
- Authorization Server

![oauth_flow](oauth_abstract_flow.png)

## Registration

User must register with the client app with the service through the service's developer portal, and provide the following:

- Application Name
- Application Website
- Redirect URI or Callback URL: where the service will redirect the user after they authorize (or deny) your application

## Grant

OAuth 2.0 defines 4 authorization mode

- Authorization Code: most commonly used. When user agrees to authorize, the application gets a code, then the application uses that code to apply with authorization server to get a token.

  ![auth_code](auth_code_flow.png)

- Implicit: The implicit grant type is used for mobile apps and web applications (i.e. applications that run in a web browser), where the client secret confidentiality is not guaranteed. The access token is given to the user-agent to forward to the application, so it may be exposed to the user and other applications on the user’s device. Also, this flow does not authenticate the identity of the application, and relies on the redirect URI (that was registered with the service) to serve this purpose.

  The process is similar to the above, but without getting a code, just getting a token directly.

  ![implicit](implicit_flow.png)

- Resource Owner Password Credentials: used with trusted Applications, such as those owned by the service itself. User surrender username and password to the application, and application use them to get authorization)

- Client Credentials: used with Applications API access. The client credentials grant type provides an application a way to access its own service account. Examples include if an application wants to update its registered description or redirect URI, or access other data stored in its service account via the API.

## Refresh token

If a refresh token was included when the original access token was issued,
eg.

```
https://cloud.digitalocean.com/v1/oauth/token?grant_type=refresh_token&client_id=CLIENT_ID&client_secret=CLIENT_SECRET
```

# Tips

Just read online to always do rebase when pull from git. Default behavior is merge.

```
git pull --rebase
```

This can be set into git config

```
git config --global pull.rebase true
```

[More granularity control](https://stackoverflow.com/questions/13846300/how-to-make-git-pull-use-rebase-by-default-for-all-my-repositories)

```
// only set a certain branch
git config branch.<name>.rebase true

// [always, never, remote, local]
git config branch.autosetuprebase always

```

# Share

## ngrok

Ngrok provides a service that can expose your web app with a resolvable name.

1. First, you need to build a web server locally listening at a port, eg. 3000.

2. Then start ngrok.

```
ngrok http 3000
```

Ngrok returns you a url resolvable from outside eg. `abcdefg.ngrok.io`.

3. Your service can be accessed through `abcdefg.ngrok.io`.

Why you need ngrok?

One use case is, you are building an app A, which hooks with another public app B, before you can publish your app A, you would like to do some integration test with B, and B can only access your app through a public resolvable name.

Ngrok has free plans and paid plans. From netstat's result, ngrok starts a host in aws to tunnel your service out, this cost something to maintain.

In the free plan, only one ngrok instance is allowed, and the domain name it returns is a random string. Every time you stop and restart, it returns a different subdomain name.

With the paid plan, you cannot appoint your subdomain name. Not sure what would happen if several services is using the same subdomain name.
