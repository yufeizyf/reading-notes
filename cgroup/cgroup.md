Control Groups
==============
cgroups是linux内核提供的一种机制，这种机制可以用来限制、记录、隔离单个进程或多个进程对物理资源的使用。

###cgroup的相关概念
* Task（任务）：cgroups中，task指系统的一个进程。
* Control Group（控制族群）：一组以某种标准划分的进程。cgroups中的资源限制的单位是控制族群
* Hierarchy（层级）：控制族群的组织形式是Hierarchy形式。
* Subsystem（子系统）：一个子系统就是一个资源控制器。cgroups为每种可以控制的资源定义了一个子系统。子系统有cpu，cupacct，cpuset，memory，blkio，devices，net_cls，freezer，ns。当子系统attach到一个Hierarchy上时，便可以对这个Hierarchy中的控制族群进行资源控制。

###子系统介绍
1. cpu：限制task（进程）的cpu使用率。
2. cpuacct：生成cgroup中task（进程）对cpu的使用报告。
3. cpuset：可以为cgroup中的task（进程）分配cpu节点或内存节点。
4. blkio：限制cgroup中的task（进程）对块设备（磁盘，USB等）的io。
5. memory：限制cgroup中task（进程）对内存的使用。
6. devices：限制cgroup中的task（进程）对设备的访问。<font color=red>（？？？？设备的概念）</font>
7. freezer：挂起或恢复cgroup中的task（进程）。
8. net_cls：使用classid（参考tc）标记网络数据包（packet），然后tc（Traffic Control）根据classid对数据爆进行控制。
9. ns：是不同的cgroup中的task（进程）使用不同的namespace。<font color=red>（？？？？？）</font>