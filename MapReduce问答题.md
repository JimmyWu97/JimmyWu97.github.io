# MapReduce问答题

#### 一、描述一下手写MR的大概流程和规范

​		创建并继承Hadoop的Mapper、Reducer类和一个Driver类

mapper类：继承Hadoop的mapper类设置其四个泛型参数，前两个参数类型为Longwritable和Text 重写其map方法，按行读取文件中的数据，通过自定义的逻辑处理后，写出处理后的key和value

reducer类：继承Hadoop的reducer类也是四个泛型参数，前两个参数类型和mapper类的输出的key和value参数类型一样，后两个参数根据用户需求定。重写reduce方法，遍历一组相同key的values，获取到处理后要输出的value。

driver类：

​				创建一个Configuration对象；

​				创建一个job对象；

​				指定当前MR的Mapper和Reducer；

​				指定当前MR的Map阶段输出的key和value的类型；

​				指定当前MR的Map阶段最终输出的key和value的类型；

​				指定输入输出路径；

​				提交job

#### 二、如何实现Hadoop中的序列化，以及Hadoop的序列化和Java的序列化有什么区别？

​		Hadoop的序列化：是一个轻量级的序列化，只需要自定bean对象实现Writable序列化接口，通过重写接口中的write和readFields方法实现对象的序列化和反序列化

​		与Java序列化相比较能够高效使用存储空间，提高读写效率，支持多语言的交互

#### 三、概述一下MR程序的执行流程

​		job的提交阶段和执行阶段：

​		1.提交阶段对MR进行初始化工作，对MT和RT进行规划

​		2.执行阶段执行每一个MT和RT

<img src="C:\Users\m1316\AppData\Roaming\Typora\typora-user-images\1665208151519.png" alt="1665208151519" style="zoom:50%;" />	

​		Input阶段通过InputFormat组件对文件进行切片，按行读取数据，规划MT数量，然后到达Mapper阶段，按行读取数据，通过自定义的业务逻辑处理好每一组<key,value>，写入环形缓冲区内，然后经过Shuffle阶段，将Map输出进行分区，分组，排序等处理后传给Reduce端。之后在Reduce再进行一次归并排序，再按照相同key进行分组，然后进入到Reducer阶段开始执行自定义业务逻辑，最后通过OutputFormat组件进行输出。

#### 四、InputFormat负责数据写的时候要进行切片，为什么切片大小默认是128M

​		因为块大小默认是128M，一个切片分配一个MapTask，如果切片大小等于块大小就不用跨节点读取数据提升MR的效率。

#### 五、描述一下切片的逻辑（从源码角度描述）

​		首先，切片发生job的提交阶段，通过InputFormat组件实现对文件进行切片，首先会遍历输入的目录，对于目录中的子目录是否忽略，默认是不忽略的；然后，判断当前文件是否可以切分，目的主要是针对压缩文件的判断，有些不支持切分的压缩文件就当成一个切片处理；然后会计算切片的大小，默认切片的大小等于块大小，也可以通过修改minSize和maxSize调整切片的大小；对于剩余的文件是否继续切分，如果剩余文件大小大于切片大小的1.1倍就继续切分。

#### 六、CombineTextInputFormat机制是怎么实现的

​		主要分为虚拟存储和切片两个过程，要先设置maxInputSplitSize。

​		虚拟存储：如果文件小于设置的切片大小划分为一个块，文件大于切片大小且小于切片大小的二倍则一分二，文件大于切片大小的二倍则先分出一个切片剩余的再一分为二。

​		切片过程：如果虚拟存储文件大小大于等于切片大小则形成一个切片，如果不大于则跟下一个虚拟存储文件合并。

#### 七、阐述一下 Shuffle机制 流程？

​		map方法输出的数据存储到环形缓冲区，缓冲区是默认大小为100M，从中间开始向两边存储，一边存放<k,v>，一边存放kv元数据和所属分区编号，当存储量到达数组的80%第一次溢写，反向存储，读取和溢写同时进行，溢写之前会对数据进行快速排序，如果发生多次溢写，会对溢写的数据进行归并排序后输出落盘

​		reduce端将数据从磁盘中拷贝到内存缓冲区里，当内存不够（到达66%）时溢写到磁盘中，对每个map来的数据归并排序，按照相同key进行分组，然后送到Reduce方法。

#### 八、在MR程序中由谁来决定分区的数量，哪个阶段环节会开始往分区中写数据？

​		分区数量由ReduceTask决定，默认情况有一个0号分区，数据写入到环形缓冲区时已经记录分区编号了，数据从环形缓冲区溢写出后进行分区。

#### 九、阐述MR中实现分区的思路（从源码角度分析）

​		首先获取当前MR的ReduceTask的值，然后看它是否大于1，如果大于1配置项没有配置就走默认分区规则，用当前key的哈希值取整后和ReduceTask的数量取余算出分区编号；或者根据业务需求自定义分区规则，继承Partitioner自定义分区器。如果当前只有一个ReduceTask，只有一个分区，Hadoop就会创建一个分区器对象Partitioner，然后获取到当前的分区编号。

#### 十、描述一下Hadoop中实现排序比较的规则

​		第一种是实现WritableComparable接口，实现接口中的compareTo方法，在方法中自定义比较规则，当程序运行时Hadoop会自动生成比较器对象。

​		第二种是自定义比较器对象需要继承Hadoop提供的WritableComparator类，重写该类中的compare方法，在无参构造中通过调用父类的super方法将自定义的比较器对象和比较的对象进行关联，最后在Driver类中指定自定义的比较器对象。

#### 十一、Hadoop中实现排序的两种方案分别是什么？

​		第一种是直接让参与比较的对象实现WritableComparable接口，并指定泛型，然后实现compareTo方法实现比较规则，Hadoop在程序运行时会自动生成比较器对象

​		第二种自定义比较器对象，创建自定义比较器类继承WritableComparator类，在构造方法中调用父类super方法将自定义的比较器对象和比较的对象进行关联，重写该类中的compare方法，自定义比较规则；自定义的Bean对象必须实现WritableComparatable接口

#### 十二、编写MR的时候什么情况下使用Combiner 具体实现流程是什么？

​		当map阶段数据量非常大时候，使用Combiner提前对数据进行reduce汇总，减轻ReduceTask的压力，减少IO的开销

​		实现流程：自定义一个Combiner类，继承Hadoop提供的Reducer，在job中指定自定义的Combiner类

#### 十三、OutputFormat自定义实现流程描述一下

​		首先，自定义一个OutputFormat类，继承Hadoop提供的OutputFormat,在该类中实现getRedcordWriter(),返回一个RecordWriter。自定义一个RecordWriter对象继承Hadoop提供的RecordWriter类，在该类中重写write()和close()方法，在方法中实现自定义输出。

#### 十四、MR实现 ReduceJoin 的思路，以及ReduceJoin方案有哪些不足？

​		首先找到表之间的关联字段；在Map阶段对表中的数据按行读取进行数据整合，关联的字段作为输出数据的key，整合过程中要标记数据来源于哪个表；在Reduce阶段，遍历一组相同key的values根据标记把数据分离出来，分别放到各自的对象中维护，然后把当前维护好的一组数据进行关联操作，得到想要的数据结果。

​		不足：数据合并的操作在Reduce阶段完成，Reduce端的处理压力太大，Map端的资源利用率不高，而且在Reduce阶段很容易发生数据倾斜问题。

#### 十五、MR实现 MapJoin 的思路，以及MapJoin的局限性是什么？

​		首先要找到表之间的关联字段，然后将小文件的数据映射到内存中的一个容器维护起来。当MapTask处理大文件的数据时，每读取一行数据，就根据当前行中的关联字段到内存的容器里获取对象的信息。最后再封装结果将其输出。

​		局限性：MapJoin只适用于一张很小的表和一张很大的表

​		