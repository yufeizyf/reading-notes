Control Groups
==============
cgroups是linux内核提供的一种机制，这种机制可以用来限制、记录、隔离单个进程或多个进程对物理资源的使用。

###cgroup的相关概念
* Task（任务）：cgroups中，task指系统的一个进程。
* Control Group（控制族群）：一组以某种标准划分的进程。cgroups中的资源限制的单位是控制族群
* Hierarchy（层级）：控制族群的组织形式是Hierarchy形式。
* Subsystem（子系统）：一个子系统就是一个资源控制器。cgroups为每种可以控制的资源定义了一个子系统。子系统有cpu，cupacct，cpuset，memory，blkio，devices，net_cls，freezer，ns。当子系统attach到一个Hierarchy上时，便可以对这个Hierarchy中的控制族群进行资源控制。

###子系统介绍
* cpu：限制task（进程）的cpu使用率。  
* cpuacct：生成cgroup中task（进程）对cpu的使用报告。
* cpuset：可以为cgroup中的task（进程）分配cpu节点或内存节点。
* blkio：限制cgroup中的task（进程）对块设备（磁盘，USB等）的io。
* memory：限制cgroup中task（进程）对内存的使用。
* devices：限制cgroup中的task（进程）对设备的访问。<font color=red>（？？？？设备的概念）</font>
* freezer：挂起或恢复cgroup中的task（进程）。
* net_cls：使用classid（参考tc）标记网络数据包（packet），然后tc（Traffic Control）根据classid对数据爆进行控制。
* ns：是不同的cgroup中的task（进程）使用不同的namespace。<font color=red>（？？？？？）</font>

###cgroup的几个性质
1. cgroup以一个伪文件系统的形式实现，用户可以通过文件操作的方式实现对cgroups的管理和操作。
2. cgroups的组织管理操作单元可以细粒度到线程级别,可以对线程进行资源限制。
3. 所有资源管理的功能都以“subsystem（子系统）”的方式实现。
4. 子进程创建之初与其父进程处于同一个cgroups的控制族群。

###cgroup各组成部分之间的关系

* 同一个Hierarchy可以附加`一个或多个`Subsystem。
* 一个Subsystem可以附加到多个Hierarchy，当且仅当这些Hierarchy只有这唯一一个Subsystem。
* 对于每一个创建的Hierarchy，task只能存在于其中一个cgroup中，即一个task不能存在于同一个hierarchy的不同cgroup中，但是一个task可以存在在不同hierarchy中的多个cgroup中。
* 进程（task）在fork自身时创建的子任务（child task）默认与原task在同一个cgroup中，但是child task允许被移动到不同的cgroup中。即`fork完成后，父子进程间是完全独立的`。


