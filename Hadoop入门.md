第一阶段:语言基础

---Java

第二阶段:大数据的基础阶段

---工具类

​	---Maven

​				--Linux

​				--Shell

--基础框架类

​	--Hadoop

​				--解决海量数据的存储(HDFS)

​						--NameNode(nn):管理真实数据的元数据

​						--DataNode(dn):对真实数据的存储

​						--SecondaryNameNode(2nn):为NameNode进行一些数据备份,恢复数据会用到,但不能保证数据完全恢复

​				--解决海量数据的计算(MapReduce)

​						--Map阶段:把需要计算的数据按照需求分成多个MapTask任务来执行

​						--Reduce阶段:把Map阶段处理完的结果拷贝过来根据需求进行汇总

​				--解决资源调度(Yarn)

​						--ResourceManager(rm):统筹管理每一台机器上的资源,并且负责接受处理客户端作业请求

​							--NodeManager(nm):负责单独每一台机器的资源管理,实时保证和rm通信

​							--ApplicationMaster:针对每个请求job(MR任务)的抽象封装

​							--Container:将来运行在YARN上的每一个任务都会给其分配资源,Container就是当前任务所需资源的抽象封装

Hadoop的优势

--高可靠性

--高扩展性

--高效性

--高容错性

