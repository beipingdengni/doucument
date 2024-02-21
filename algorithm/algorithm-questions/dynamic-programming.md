

# 动态规划

## 硬币找零问题

https://blog.csdn.net/flying_all/article/details/99706686

### 计算多少种组合（最少要用多少张纸币）

```java
public class MinCoins {
    public static void main(String[] args) {
        int[] coins = {1, 3, 5}; // 不同币值的硬币
        int amount = 9; // 要支付的金额

        int minCoins = minCoinsNeeded(coins, amount);
        System.out.println("支付 " + amount + " 元，最少需要 " + minCoins + " 个硬币。");
    }

    public static int minCoinsNeeded(int[] coins, int amount) {
        int max = amount + 1;
        int[] dp = new int[max];
        // 初始化数组，除了dp[0]为0，其余都设置为一个较大的数，表示初始状态下无法用硬币组合出该金额
        Arrays.fill(dp, max);
        dp[0] = 0;

        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
        }
        // 如果dp[amount]的值没有被更新，说明无法组合出该金额，返回-1或者其他标识错误的值
        return dp[amount] > amount ? -1 : dp[amount];
    }
}

```

### 当硬币有数量限制时，问题变成了一个组合优化问题，我们不能再简单地使用动态规划中的无限硬币的方法。我们需要考虑每种硬币数量的限制，这个问题可以使用“有限背包问题”的方案来解决

```java
public class MinCoinsWithLimit {
    private static int minCoins = Integer.MAX_VALUE; // 存储结果的全局变量

    public static void main(String[] args) {
        int[] coins = {1, 3, 5}; // 不同币值的硬币
        int[] counts = {5, 2, 2}; // 对应币值硬币的数量限制
        int amount = 11; // 要支付的金额

        findMinCoins(coins, counts, amount, 0, 0);
        System.out.println("支付 " + amount + " 元，最少需要 " + (minCoins == Integer.MAX_VALUE ? -1 : minCoins) + " 个硬币。");
    }

    public static void findMinCoins(int[] coins, int[] counts, int remaining, int start, int usedCoins) {
        if (remaining == 0) { // 如果剩余金额为0，更新最小硬币数
            minCoins = Math.min(minCoins, usedCoins);
            return;
        }
        if (start == coins.length) { // 如果没有更多的硬币可以选择，返回
            return;
        }

        for (int k = 0; k <= counts[start] && k * coins[start] <= remaining; k++) {
            // 尝试使用0到counts[start]个coins[start]面值的硬币
            findMinCoins(coins, counts, remaining - k * coins[start], start + 1, usedCoins + k);
        }
    }
}

```



### 计算多少种组合内容

```java
import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;

public class CoinChangeCombinations {

    public static void main(String[] args) {
        int[] coins = {1, 2, 5}; // 面值数组
        int amount = 5; // 目标金额

        // 按照面值从大到小对硬币数组进行排序
        Arrays.sort(coins);
        int[] reversedCoins = new int[coins.length];
        for (int i = 0; i < coins.length; i++) {
            reversedCoins[i] = coins[coins.length - 1 - i];
        }
      
        List<List<Integer>> result = new ArrayList<>();
        coinChangeCombinations(reversedCoins, amount, new ArrayList<>(), result, 0);
        System.out.println("所有可能的组合编号为（面值大的在前）: ");
        for (List<Integer> combination : result) {
            System.out.println(combination);
        }
    }

    public static void coinChangeCombinations(int[] coins, int amount, List<Integer> currentCombination,
                                              List<List<Integer>> result, int startIndex) {
        if (amount == 0) {
            result.add(new ArrayList<>(currentCombination));
            return;
        }

        for (int i = startIndex; i < coins.length; i++) {
            if (amount - coins[i] >= 0) {
                currentCombination.add(coins[i]);
                coinChangeCombinations(coins, amount - coins[i], currentCombination, result, i);
                currentCombination.remove(currentCombination.size() - 1);
            }
        }
    }
}
```

