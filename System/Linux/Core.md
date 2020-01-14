# 线程,进程,进程组和会话

* 区分
  * 一个**会话**本身就是一个进程组(又称**会话进程组**), 该进程组又可以划分为多个进程组.
  * 一个**进程组**可以含有多个进程
  * 一个**进程**可含多个**线程**

* ID

  上述每种对象都有自己的ID, 如线程号，进程号，进程组号和会话号.

  * 一般情况下,  一个进程只含一个线程, 且进程号与线程号一致
  * 会话和进程组中, 第一个被创建的进程称为**Leader**, 该Leader的进程号就是会话或进程组的ID

* 进程组类型与会话的关系

  进程组分为**前台进程组**、**后台进程组**和**孤儿进程组**（Orphaned process groups）, 概念上类似于Bash的前台任务, 后台任务和Bash.

  * 会话进程组本身就是孤儿进程组; 会话进程组可划分为前台进程组和后台进程组; 会话最多拥有一个前台进程组, 可拥有多个后台进程组.
  
    > 孤儿进程组到底有什么区别? 
    >
    > 不太清除, 我猜父进程是init进程的进程是会话(孤儿)Leader ( 我通过pstree观察的结论 ). 
    >
    > 若手动让进程**守护进程化**(孤儿进程), 最好分离TTY, 否则会收到终端的停止信号.
  
  * 每个会话都关联一个**控制终端**（controlling tty）, 会话中进程被创建时将继承该终端, 因此都可以通过**流**来访问终端
  
  * 前台进程组的三种标准流都能正常使用, 达到与用户交互的目的
  
  * 后台进程组不能使用标准输入流, 否则将收到停止信号.
  
* 进程结束

  * 会话Leader结束后, 将发送结束信号给前台进程组的所有进程; 发送结束信号给后台进程组的所有进程, 接着又发送继续信号.
    * 子进程可捕获或忽略掉结束信号, 该进程及其子进程将组成孤儿进程组, 该进程成为Leader
    * 结束掉的子进程的子进程也将形成单个的孤儿进程组.

  * 普通进程结束后, 其子进程直接成为孤儿进程组Leader吧?

--------------------
其实我理解的不够，上面的解释我自己都是懵懵的，**很有可能有错误**。但结合实践来说，我想说明的是这几点：
* jobs应该是对应着这里的进程组，在shell上可以执行一个jobs，也就是进程组，比如通过重定向连接多个程序形成进程组。单个进程也算进程组吧。
* 每个ssh连接就是一个会话，关闭ssh，在ssh上运行的程序基本会停止。
* ~~想要常驻内存，成为守护进程，需要解除进程与控制终端的绑定 ( 猜测, 待确认 )~~。
* 每个会话都会连接一个控制终端，但控制终端对应的具体设备文件名或许不知道，但是`/dev/tty`永远指着当前会话的控制终端，对它操作就是了。`/dev/tty`是指向当前会话的终端，即使你开多个会话，两个会话的当前控制终端也是不同的，即tty是动态生成数据的。

>参考：
>
>* https://www.win.tue.nl/~aeb/linux/lk/lk-10.html
>* https://www.gnu.org/software/libc/manual/html_node/Controlling-Terminal.html
>* https://unix.stackexchange.com/questions/404555/what-is-the-purpose-of-the-controlling-terminal



# 待完善

* 用户管理

  * 登录记录

    https://blog.csdn.net/xiaotangxianshengqi/article/details/82687701

  * `last`

  * `whoami`

# 参考

* [《How-Linux-Works-2nd-Edition》](https://github.com/KnowNo/How-Linux-Works-2nd-Edition)
* [linux course](https://geek-university.com/linux/what-is-linux/)
* [The Linux Command Line](http://linuxcommand.org/index.php)
* [vim tutorial](https://www.tutorialspoint.com/vim/index.htm)

----------

# 待学习

* [Linux学习教程，Linux入门教程（超详细）](http://c.biancheng.net/linux_tutorial/)