

- synchronized是悲观锁，这种线程一旦得到锁，其他需要锁的线程就挂起的情况就是悲观锁。

- CAS操作的就是乐观锁，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。





学习总结：

cas compare and swap 要修改前，先读取待修改的数据的值，存储为期望值，然后这边计算下，要修改后的值，修改后是什么值，然后进行比较，将我们前面存储的期望值和现在的数据的值，进行对比，如果一样，说明没有其他线程过来修改过这个数据，这时候，我们就可以把我们计算的值赋值给这个我们修改的值，如果这个值和期望值不一样，那么说明这个值已经被修改了，这时候，我们就不能修改了，这时，我们的修改请求不会中断，还会继续上面的过程，再获取新的数据的值，存储期望值，重复上面操作，直到成功为止，

do{

存储旧的期望值，

生成修改后的值

}while(CompareAndSwap(期望值，现在数据的值，修改后的值))

只有这个修改成功后，才会退出。**这个重新尝试的过程被称为自旋。**



思想

悲观锁：Synchronized属于悲观锁，悲观的认为程序中并发情况严重，所以严防死守。

乐观锁：CAS属于乐观锁，乐观的认为程序中的并发情况不那么严重，所以让线程不断去尝试更新。



可以理解成一个无阻塞多线程争抢资源的模型。



CAS的缺点：



CPU开销大：

因为如果请求修改不成功，就会一直去重试，这样会循环反复，会给CPU带来很大的压力。

不能保证代码块的原子性：

操作只能一个变量，如果有多个变量同时需要保证，那么就只能使用synchronized去处理代码块。





但是这种得不到状态为true时使用递归算法是很耗cpu资源的，所以一般情况下，都会有线程sleep。

CAS的缺点：

1.CPU开销较大
 在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给CPU带来很大的压力。

2.不能保证代码块的原子性
 CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。比如需要保证3个变量共同进行原子性的更新，就不得不使用Synchronized了。



作者：AubreyXue
链接：https://www.jianshu.com/p/ae25eb3cfb5d
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。C