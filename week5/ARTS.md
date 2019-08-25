# Algorithm

### coin change

https://leetcode.com/problems/coin-change/

In a special case when coins are provided in some certain combinations, greedy algorithm can be applied, but for a general solution, dp should be used.

In general, dp remembers sub solution to reduce time when compared to backtrace.

```
func coinChange(coins []int, amount int) int {
    dp:=make([]int, amount+1)
    for i:=1; i <=amount; i++{
        dp[i] = amount + 1
    }
    for i:=1; i <=amount; i++{
        for j:=0; j < len(coins); j++{
            if i >= coins[j]{
                dp[i] = min(dp[i],dp[i - coins[j]] + 1)
            }
        }
    }
    if dp[amount] == amount + 1{
        return -1
    }
    return dp[amount]
}

func min(i, j int) int{
    if i < j {
        return i
    }
    return j
}
```

Turns out go does not have min for int, only for float.

### coin change 2

This one asks for how many combinations. The tricky part

- to not duplicate, the outer loop need to be coins. This can be deducted by drawing it out using a small number.
- dp[0] = 1, meaning 0 amount has 1 combination.

```
func change(amount int, coins []int) int {
    dp:=make([]int, amount+1)
    dp[0] = 1

    for j:=0; j < len(coins); j++{
        for i:=1; i <=amount;i++{
            if i >= coins[j]{
                dp[i] += dp[i-coins[j]]
            }
        }
    }
    return dp[amount]
}
```

# Review

# Tips

### Most useful tips found during the week

### https://github.com/frekele/oracle-java

download oracle jdk can refer to that link and no need to waste time registering an account. oracle 的用户体验真是一如既往地值得吐槽

# Share
