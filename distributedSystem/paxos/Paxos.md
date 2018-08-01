## Paxos ##
### Paxos是什么 ###
* Paxos 算法解决的问题是一个分布式系统如何就某个值（决议）达成一致
* 一个典型的场景是，在一个分布式数据库系统中，如果各节点的初始状态一致
* **每个节点都执行相同的操作序列，那么他们最后能得到一个一致的状态**
* 为保证每个节点执行相同的命令序列，需要在每一条指令上执行一个“一致性算法”以保证每个节点看到的指令一致
* <font color=red>**关键是，保证每个节点得到的信息一致！**</font>

### Paxos的两个原则 ###
* 安全原则--保证不能做错的事
	* 只能有一个值被批准，不能出现第二个值把第一个覆盖的情况
	* 每个节点只能学习到已经被批准的值，不能学习没有被批准的值

* 存活原则---只要有多数服务器存活并且彼此间可以通信最终都要做到的事
	* 最终会批准某个被提议的值
	* 一个值被批准了，其他服务器最终会学习到这个值

* <font color=red>**关键：只有一个proposer时，肯定会被批准**</font>
* <font color=red>**只有当对一个对象(logID)有多个提交时，才进行选择**</font>
* 如果两个实例分配了同一个logID，并且一个被选择了，另一个实例，则要增大logID值，重新进行paxos，直到被选择上

### Paxos的两个组件 ###
* Proposer：提议发起者，处理客户端请求，将客户端的请求发送到集群中，以便决定这个值是否可以被批准
* Acceptor：提议批准者，负责处理接收到的提议，他们的回复就是一次投票。会存储一些状态来决定是否接收一个值

### 二段提交原则 ###
* 优先检查有没有已经被批准的值
* 如果有应该提议已经被批准的值而放弃自己的值
* **问题：两个实例在判断是都没有批准，则会批准两次**
	* 可能后一次又形成了多数派

### 二段提交+提议排序 ###
* 解决多次批准的问题：一旦Acceptor批准了某个值，其他有冲突的值都应该被拒绝
* 需要对Proposer进行排序，将排序在前的赋予高优先级，Acceptor批准优先级高的值，拒绝排在后的值
* 为每个提议赋予一个唯一ID，规定这个ID越大，优先级越高
* 针对同一个提议，提议者发送提议之前，就需要生成一个唯一的ID，而且需要比之前使用的或者生成的都要大

### 提议ID生成算法 ###
* 假设有n个proposer，每个编号为ir(0<=ir<n)
* proposor编号的任何值s都应该大于它已知的最大值，并且满足：s %n = ir 
* s = m*n + ir

以3个proposer P1、P2、P3为例，开始m=0,编号分别为0，1，2：  

* P1提交的时候发现了P2已经**提交**(没有被批准)，P2编号为1 > P1的0，因此P1重新计算编号：new P1 = 1*3+0 = 4
* P3以编号2提交，发现小于P1的4，因此P3重新编号：new P3 = 1*3+2 = 5


