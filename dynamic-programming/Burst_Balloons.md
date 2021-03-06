题目描述
========================
有 n 个气球，编号为0 到 n-1，每个气球上都标有一个数字，这些数字存在数组 nums 中。
现在要求你戳破所有的气球。每当你戳破一个气球 i 时，你可以获得 nums[left] * nums[i] * nums[right] 个硬币。 这里的 left 和 right 代表和 i 相邻的两个气球的序号。注意当你戳破了气球 i 后，气球 left 和气球 right 就变成了相邻的气球。
求所能获得硬币的最大数量。

说明:

  你可以假设 nums[-1] = nums[n] = 1，但注意它们不是真实存在的所以并不能被戳破。
	0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100


示例:

输入: [3,1,5,8]
输出: 167 
解释: nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
     coins =  3* 1 * 5      +  3* 5 * 8    +  1* 3 * 8      + 1* 8 * 1   = 167
 
解法
========================
Ⅰ分治法
--------------------------
　在使用分治法时，我们应该考虑的核心问题是如何用子问题的解来表示原问题的解，也就是子问题该如何划分才能通过子问题来求解原问题。我们把描述子问题的解与原问题的解之间的关系的表达式称为状态转移方程。

　　首先我们尝试每戳破一个气球，以该气球为边界将气球数组分为两部分，使用这两部分的解来求解原问题。

　　我们设戳破区间 i 到 j 间的气球我们得到的最大金币数为coin。及coin = def( i , j )。

　　则当我们戳破气球 k 时，两边区间的最大值分别是  def( i , k-1 ) 与 def( k+1 , j )。

　　此时我们发现了问题，因为戳破了气球 k ，气球数组的相邻关系发生了改变，k-1 与 k+1 原本都与 k 相邻，而 k 戳破后他们两个直接相邻了。而且先戳破 k+1 与先戳破 k-1 得到的结果将完全不同，也就是说两个子问题间发生了依赖。如果先戳破 k-1 ，则 k+1 左边的相邻气球变成了 k-2；反之 k-1 右边相邻的气球变成了 k+2 。

　　子问题的处理顺序将影响到每个子问题的解，这将使我们的状态转移方程极为复杂和低效，我们应当换一种划分子问题的方式，使每个子问题都是独立的。

　　那么我们换一种划分方式，既然两个子问题都依赖 k 和两个边界，那么我们划分子问题时，k 与两个边界的气球我们都不戳破，求出 i+1 到 k-1 与 k+1 到 j-1 之间的解。这样两个子问题间的依赖便被消除了，两个边界及气球 k 不被戳破，两个子问题的依赖都不会越过 k 到另一个子问题上，子问题间是相互独立的。

　　并且在两个子问题解决后，气球序列还剩下 k 与两个边界的气球没有戳破，那么我们用两个子问题的解与戳破 k 与两个边界的最大值即可求出原问题的解。

　　那么 def( i , j ) 函数的定义则为，不戳破 i 与 j ，仅戳破 i 与 j 之间的气球我们能得到的最大金币数。

　　如此划分，状态转移方程为： def( i, j ) = def( i , k ) + def( k , j )+nums[ i ][ j ][ k ]

　　其中 nums[ i ][ j ][ k ] 为戳破气球 k 时我们能得到的金币数，因为def( i , j )表示戳破 i 到 j 之间的气球，自然包括 k 。

　　上述方程其实还有问题，前面说过，为了保证我们可以完整的搜索解空间，我们需要尝试所有的子问题划分方式，对于上述状态转移方程，也就是 k 的取值。k 的取值应当介于 i+1 与 j-1 之间，我们尝试所有 k 的取值并从中挑选最大值，这才是原问题真正的解。

　　真正的状态转移方程应该为：def( i, j ) = max { def( i , k ) + def( k , j )+nums[ i ][ j ][ k ] } | i<k<j

　　这样我们便找到了用子问题的解来表示原问题的解的方法，或者说子问题的划分方式。因为我们要划分子问题，必然不是只划分一次这么简单。而是要把问题一直划分到不能继续划分，也就是划分到问题规模最小的最小子问题，使效率最大化。

　　因为 k 是介于 i 与 j 之间的，那么当 i 与 j 相邻时我们的问题将不能再继续划分。此时按照我们对问题的定义，“不戳破 i 与 j ，仅戳破 i 与 j 之间的气球”，因为 i 与 j 之间没有气球，我们得到的金币数是 0 。

　　为了保证问题定义的正确性，我们向上推演一次。def( i , i+2 ) = def( i , i+1 ) + def( i+1 , i+2 ) + nums[i]*nums[ i+1]*nums[i+2]

　　def( i , i+1 ) , def( i+1 , i+2 ) 都是最小子问题，返回0。即 def( i , i+2 ) =  nums[i]*nums[ i+1]*nums[i+2] 。因为问题的定义我们不戳破 i 与 i+2，所以我们只能戳破 i+1，戳破 i+1得到的金币确实是 nums[i]*nums[ i+1]*nums[i+2] 即  def( i , i+2 )  。

　　所以说对于我们的状态转移方程 def( i, j ) = max { def( i , k ) + def( k , j )+nums[ i ][ j ][ k ] } | i<k<j ，回归条件 def( i , i+1 ) = 0 是正确的。

　　状态转移方程与回归条件都找到了，实现起来就很简单了：
  ```java
  /**
    *   @Author Nyr
    *   @Date 2019/11/30 0:23
    *   @Param  nums:气球数组；length:数组长度，避免每层都计算一次；begin:开始下标；end:结束下标；cache:缓存，避免重复计算
    *   @Return 
    *   @Exception 
    *   @Description 状态转移方程 def( i, j ) = max { def( i , k ) + def( k , j )+nums[ i ][ j ][ k ] } | i<k<j 的实现
    */
    public static int maxCoins4(int[] nums, int length, int begin, int end,int[][] cache) {
        //回归条件，问题分解到最小子问题
        if (begin == end - 1) {
            return 0;
        }
        //缓存，避免重复计算
        if(cache[begin][end]!=0){
            return cache[begin][end];
        }
        //维护一个最大值
        int max = 0;
        //状态转移方程 def( i, j ) = max { def( i , k ) + def( k , j )+nums[ i ][ j ][ k ] } | i<k<j
        for (int i = begin + 1; i < end; i++) {
            int temp = maxCoins4(nums, length, begin, i,cache) +
                    maxCoins4(nums, length, i, end,cache) + nums[begin] * nums[i] * nums[end];
            if (temp > max) {
                max = temp;
            }
        }
        //缓存，避免重复计算
        cache[begin][end]=max;
        return max;
    }
```    
我们再封装一层方法，对空数组进行处理。因为 def( i , j ) 并不戳破两个边界的气球，我们为气球数组加上虚拟的边界：
```java
public static final int maxCoins4MS(int[] nums) {
        //空数组处理
        if (nums == null) {
            return maxCoin;
        }
        //加虚拟边界
        int length = nums.length;
        int[] nums2=new int[length+2];
        System.arraycopy(nums,0,nums2,1,length);
        nums2[0]=1;
        nums2[length+1]=1;
        length=nums2.length;
        //创建缓存数组
        int[][] cache=new int[length][length];
        //调用分治函数
        return maxCoins4M(nums2, length,cache);
    }

    public static int maxCoins4M(int[] nums, int length,int[][] cache) {
        int max = maxCoins4(nums, length, 0, length - 1,cache);
        return max;
    }
```
Ⅱ动态规划，状态转移方程
------------------------------------
　动态规划

　　动态规划算法通常用于求解具有某种最优性质的问题。在这类问题中，可能会有许多可行解。每一个解都对应于一个值，我们希望找到具有最优值的解。动态规划算法与分治法类似，其基本思想也是将待求解问题分解成若干个子问题，先求解子问题，然后从这些子问题的解得到原问题的解。与分治法不同的是，适合于用动态规划求解的问题，经分解得到子问题往往不是互相独立的。对于分治法求解的问题，子问题的相互独立仅仅是同层级的子问题间没有互相依赖。但对于动态规划而言，同层级的子问题可能会依赖相同的低层级问题，这就导致低层级问题可能会被计算多次。

　　若用分治法来解这类问题，则分解得到的子问题数目太多，有些子问题被重复计算了很多次。如果我们能够保存已解决的子问题的答案，而在需要时再找出已求得的答案，这样就可以避免大量的重复计算，节省时间。我们可以用一个表来记录所有已解的子问题的答案。不管该子问题以后是否被用到，只要它被计算过，就将其结果填入表中。这就是动态规划法的基本思路。具体的动态规划算法多种多样，但它们具有相同的填表格式。

　　其实在上面的分治解法，我加入了一个二维数组用于缓存已经计算过的子问题的结果，将缓存去掉才是概念上的分治解法。而加入了缓存避免了子问题的重复计算，已经是一个动态规划解法的雏形，我们只需要将递归改为递推便是动态规划解法。正如上面所说，通常情况下，递归的解法是不可以放在生产环境的，因为我们很难控制问题规模的大小，无法预料何时会有爆栈的风险。

　　具有最优子结构性质以及重叠子问题性质的问题可以通过动态规划求解。

　　最优子结构

如果一个问题的最优解包含其子问题的最优解，我们就称此问题具有最优子结构
一个问题具有最优子结构，可能使用动态规划方法，也可能使用贪心方法。所以最优子结构只是一个线索，不是看到有最优子结构就一定是用动态规划求解
　　重叠子问题

子问题空间必须足够“小”，即在不断的递归过程中，是在反复求解大量相同的子问题，而不是每次递归时都产生新的子问题。
一般的，不同子问题的总数是输入规模的多项式函数为好
如果递归算法反复求解相同的子问题，我们就称最优化问题具有重叠子问题性质
　　对于前面的分治解法，我们的计算过程分为两个阶段：

　　1、递归的不断的分解问题，直到问题不可继续分解。

　　2、当问题不可继续分解，也就是分解到最小子问题后，由最小子问题的解逐步向上回归，逐层求出上层问题的解。

　　阶段1我们称为递归过程，而阶段2我们称为递归调用的回归过程。我们要做的，就是省略递归分解子问题的过程，将阶段2用递推实现出来。

　　举个例子，对于区间 0 到 4 之间的结果，递归过程是：

　　dp[0][4] =max { dp[0][1]+`dp[1][4]`+nums[0]*nums[1]*nums[4] , dp[0][2]+`dp[2][4]`+nums[0]*nums[2]*nums[4] ,  dp[0][3]+`dp[3][4]`+nums[0]*nums[3]*nums[4] }

　　标红部分没有达到回归条件，会继续向下分解，以 `dp[1][4]` 为例：

　　dp[1][4]= max { dp[1][2]+`dp[2][4]`+nums[1]*nums[2]*nums[4] ,  `dp[1][3]`+dp[3][4]+nums[1]*nums[3]*nums[4] }

　　标红部分继续分解：

　　dp[2][4]= dp[2][3] + dp[3][4] + nums[2]*nums[3]*nums[4]

　　dp[1][3] = dp[1][2] + dp[2][3] + nums[1]*nums[2]*nums[3]

　　到这里因为已经分解到了最小子问题，最小子问题会带着它们的解向上回归，也就是说我们的回归过程是：dp[3][4] , dp[2][3] , dp[2][4] , dp[1][2] , dp[1][3] , dp[1][4] , dp[0][1] , dp[0][2] , dp[0][3] , dp[0][4] 。因为 dp[i][j] 依赖的是 dp[i][k] 与 dp[k][j] 其中 i < k < j ，也就是说如果要求解 dp[ i ][ j ] 依赖了 [ i ][ 0 ] 到 [ i ][ j-1 ] 以及 [ i+1 ][ j ] 到 [ j-1 ][ j ] 的值。那么我们在dp表中 i 从 length 递减到 0， j 从 i+1 递增到 j 推演即可。
`dp[i][j]=max(dp[i][j],dp[i][k]+dp[k][j]+nums[i]*nums[k]*num[j])`
************************************
*****************************
```c
int maxCoins(int* nums, int numsSize)
{
    if (nums == NULL || numsSize == 0) {
        return 0;
    }
    int tmpNums[numsSize + 2];
    memcpy(tmpNums + 1, nums, sizeof(int) * numsSize);
    tmpNums[0] = 1;
    tmpNums[numsSize + 1] = 1;
    int dp[numsSize + 2][numsSize + 2];
    memset(dp, 0, sizeof(dp));
    for (int len = 2; len < numsSize + 2; len++) {
        for (int i = 0; i < numsSize; i++) {
            int j = i + len;
            if (j > numsSize + 1) {
                continue;
            }
            int tmpMax = 0;
            for (int k = i + 1; k < j; k++) {
                int tmpMax = dp[i][k] + dp[k][j] + tmpNums[i] * tmpNums[k] * tmpNums[j];/*反正就是穷举，先是3个  
		连续的挨个类似滑动窗口的乘积然后再len先空1个再2个。。。。。那样跳，k在中间移动来达到第一次中间某个  
		气球被扎（依此类推）。dp[i][k]+dp[k][j]就是上一步的乘积。*/
                if (tmpMax>dp[i][j]) {
                    dp[i][j] =tmpMax ;
                }
            }
        }
    }
    return dp[0][numsSize + 1];//之所以是0是因为到最后的时候只有i=0一次循环因为tmpMAX等于上一次循环的tmpmax
}
```
### Ⅱ回溯法
```c
/**
    *   @Author Nyr
    *   @Date 2019/11/30 22:24
    *   @Param nums:气球数组，
               y：递归层级，即currentLevel,
               length：数组长度，防止每层都计算一次， 
               beforeCoins：之前所有层获得的金币和，即currentCoin
    *   @Return 
    *   @Exception 
    *   @Description  回溯解法 
    */
    public static void maxCoins2(int[] nums, int y, int length, int beforeCoins) {
        //回归条件
        if (y == length) {
            if (beforeCoins > maxCoin) {
                maxCoin = beforeCoins;
            }
            return;
        }
        for (int i = 0; i < length; i++) {
            //略过已经戳破的气球
            if (nums[i] == -1) {
                continue;
            }
            //标记已经戳破的气球
            int temp = nums[i];
            nums[i] = -1;
            //获取上一个气球的数字
            int before = i - 1;
            int beforeNum = 0;
            while (before > -1 && nums[before] == -1) {
                before--;
            }
            if (before < 0) {
                beforeNum = 1;
            } else {
                beforeNum = nums[before];
            }
            //获取下一个气球的数字
            int next = i + 1;
            int nextNum = 0;
            while (next < length && nums[next] == -1) {
                next++;
            }
            if (next > length - 1) {
                nextNum = 1;
            } else {
                nextNum = nums[next];
            }
            //计算戳破当前气球的coin
            int tempCoin = temp * nextNum * beforeNum;
            //递归进行下一戳
            maxCoins2(nums, y + 1, length, beforeCoins + 
            tempCoin);
            //回溯尝试其它戳法
            nums[i] = temp;
        }
    }
```   
