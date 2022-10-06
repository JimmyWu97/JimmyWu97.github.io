# Hadoop默认切片规则

切片发生在提交阶段

1.遍历输入文件，针对输入目录中的子目录是否进行忽略，默认不忽略

2.判断当前文件是否可以切分（目的：主要针对压缩文件的判断，有一些不支持切分的压缩文件会将其当成一个切片处理）

3.计算切片大小，默认情况切片大小等于块大小，也可以通过修改minSize和maxSize调整切片大小

	long splitSize = computeSplitSize(blockSize, minSize, maxSize);
			// 默认情况下 切片大小=块大小
			// 如果想把切片调大 修改minSize
			// 如果想把切片调小 修改maxSize
	protected long computeSplitSize(long blockSize, long minSize,long maxSize) {
			return Math.max(minSize, Math.min(maxSize, blockSize));
						   														}
	             long bytesRemaining = length;

4.剩余大小是否要继续切分，如果剩余文件大小闭上块大小大于1.1则继续切分。

