# Tinker热更新原理分析 #
1. tinker将old.apk和new.apk做了diff，拿到patch.dex，然后将patch.dex与本机中apk的classes.dex做了合并，生成新的classes.dex，运行时通过反射将合并后的dex文件放置在加载的dexElements数组的前面。
2. 找到PathClassLoader（BaseDexClassLoader）对象中的pathList对象
3. 根据pathList对象找到其中的makeDexElements方法，传入patch相关的对应的实参，返回Element[]对象
4. 拿到pathList对象中原本的dexElements方法
步骤2与步骤3中的Element[]数组进行合并，将patch相关的dex放在数组的前面
5. 最后将合并后的数组，设置给pathList

----------
 
> #合成patch##

- 首先tinker会检查相关配置和path的合法性进行检查
- 然后调用TinkerPatchService，他其实是一个IntentService好处不需要自己去new Thread了；另一方面不需要考虑在什么时候关闭该Service了。
- 通过Dexdiff算法进行合成，核心代码主要在extractDexDiffInternals中，先解析了meta里面的信息，meta中包含了patch中每个dex的相关数据。然后通过Application拿到sourceDir，其实就是本机apk的路径以及patch文件；根据mate中的信息开始遍历，其实就是取出对应的dex文件，最后通过patchDexFile对两个dex文件做合并。

----------
>##Dexdiff算法##
1. dex对Java类文件重新排列，将所有JAVA类文件中的常量池分解，消除其中的冗余信息，重新组合形成一个常量池，所有的类文件共享同一个常量池，使得相同的字符串、常量在DEX文件中只出现一次，从而减小了文件的体积
2. 一个dex文件的每个字节表示的是什么内容。对于一个类似于二进制的文件，最好的办法肯定不是靠记忆，好在有这么一个软件可以帮助我们的分析：软件名称：010 Editor
3. 首先我们猜测下header的作用，可以看到起包含了一些校验相关的字段，和整个dex文件大致区块的分布(off都为偏移量)。
4. 这样的好处就是，当虚拟机读取dex文件时，只需要读取出header部分，就可以知道dex文件的大致区块分布了；并且可以检验出该文件格式是否正确、文件是否被篡改等。
5. 算法的过程比较简单，描述一下就是：二路归并算法
首先我们需要将新旧内容排序，这需要针对排序的数组进行操作
新旧两个指针，在内容一样的时候 old、new 指针同时加1，在 old 内容小于 new 内容(注：这里所说的内容比较是单纯的内容比较比如'A'<'a')的时候 old 指针加1 标记当前 old 项为删除
在 old 内容大于 new 内容 new 指针加1， 标记当前 new 项为新增
6. 

 




