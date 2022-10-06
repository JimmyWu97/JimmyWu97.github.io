# Partitioner分区器对象

### 分区的原则：

​		分区是由需求决定的，分区编号的产生是由ReduceTask的数量控制

### Hadoop默认分区规则

		-- Hadoop默认的分区规则
		   --> 定位Mapper逻辑中的 context.write(outk, outv);
	       --> 跟进 write(outk, outv);
		   --> public void write(KEYOUT key, VALUEOUT value) 
	           throws IOException, InterruptedException;
		   --> 找具体实现 TaskInputOutputContextImpl
		        public void write(KEYOUT key, VALUEOUT value
	                ) throws IOException, InterruptedException {
					output.write(key, value);
				  }
			   
		   --> 跟进 output.write(key, value);  RecordWriter类中
		       public abstract void write(K key, V value
	           ) throws IOException, InterruptedException;
			    
		   --> 找到具体实现  NewOutputCollector
		        MapOutputCollector ： 此对象就是环形缓冲区对象
		        @Override
				public void write(K key, V value) throws IOException, InterruptedException {
				  // 为当前写入到环形缓冲区的k v 计算分区编号  partitions就是当前ReduceTask的数量
				  collector.collect(key, value,
									partitioner.getPartition(key, value, partitions));
				}
		   --> 跟进 partitioner.getPartition(key, value, partitions)
		       注意：跟进后发现来到了一个 叫做 Partitioner的抽象类中，如果想知道
			         Hadoop默认的分区规则，必须得知道 当前Partitioner的默认实现类！
					 
		   --> 关注  NewOutputCollector(org.apache.hadoop.mapreduce.JobContext jobContext,
	                   JobConf job,
	                   TaskUmbilicalProtocol umbilical,
	                   TaskReporter reporter
	                   ) throws IOException, ClassNotFoundException {
				  // 获取环形缓冲区对象
				  collector = createSortingCollector(job, reporter);
				  // 获取ReduceTask的数量
				  partitions = jobContext.getNumReduceTasks();
				  if (partitions > 1) {
				    // 如果ReduceTask的数量 大于一 走以下逻辑
					// 通过从Job的配置文件中获取一个Partitioner的实现类似的class文件
					// 在经过反射获取做种实现
					partitioner = (org.apache.hadoop.mapreduce.Partitioner<K,V>)
					  ReflectionUtils.newInstance(jobContext.getPartitionerClass(), job);
					  
					--> 跟进到 jobContext.getPartitionerClass() 
					 public Class<? extends Partitioner<?,?>> getPartitionerClass() 
						 throws ClassNotFoundException {
						return (Class<? extends Partitioner<?,?>>) 
						  // 通过配置项 mapreduce.job.partitioner.class 获取实现类的class
						  // 如果没有获取到 就使用 HashPartitioner.class 
						  // HashPartitioner 就是 Partitioner的默认实现 
						  conf.getClass(PARTITIONER_CLASS_ATTR, HashPartitioner.class);
					  }
					
				  } else {
					partitioner = new org.apache.hadoop.mapreduce.Partitioner<K,V>() {
					  @Override
					  public int getPartition(K key, V value, int numPartitions) {
						return partitions - 1;
					  }
					};
				  }
				}	
	       --> 	关注 HashPartitioner中的实现规则
		         // 根据当前的key的hashcode值和 ReduceTask的数量取余操作得到当前
					key的所属分区编号（这个规则其实没有规则的规则-有一定规律的变相随机）
	             public int getPartition(K key, V value,
										  int numReduceTasks) {
					return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
				 }		   				 
				 
	     	  --> 在MR中使用分区，通常要结合业务去做自定义分区规则！

	自定义分区器对象（结合案例）
			    
				-- ①  自定一个分区器类，继承Hadoop提供的Partitioner类，实现
				      getPartition() 方法，在方法中编写自己的业务逻辑，最终
					  给当前kv返回所属的分区编号。
					  
				-- ②  使用自定义的分区器对象
				      -- 在job提交流程中根据需求设置ReduceTask的数量
					     job.setNumReduceTasks(5);
					  -- 在job提交中指定自定义的分区器对象的class文件
						 job.setPartitionerClass(PhonePartitioner.class);
					  
			    -- ③ 分区器使用时注意事项
				     
					 -- 当ReduceTask的数量设置 > 实际用到的分区数 此时会生成空的分区文件
					 -- 当ReduceTask的数量设置 < 实际用到的分区数 此时会报错
					 -- 当ReduceTask的数量设置 = 1 结果文件会输出到一个文件中，由以下源码
					    可以论证：
						 // 获取当前ReduceTask的数量
					     partitions = jobContext.getNumReduceTasks();
						 // 判断ReduceTask的数量 是否大于1，找指定分区器对象
					     if (partitions > 1) {
							partitioner = (org.apache.hadoop.mapreduce.Partitioner<K,V>)
							  ReflectionUtils.newInstance(jobContext.getPartitionerClass(), job);
						  } else {
						    // 执行默认的分区规则，最终返回一个唯一的0号分区
							partitioner = new org.apache.hadoop.mapreduce.Partitioner<K,V>() {
							  @Override
							  public int getPartition(K key, V value, int numPartitions) {
								return partitions - 1;
							  }
							};
						  }
						  
					  -- 分区编号生成的规则：根据指定的ReduceTask的数量 从0开始，依次累加。