title: Spark简介
date: 2016-06-16 21:24:30
tags:
 - Spark
 - 并行计算
 - Python
---

##概述
这篇文章是我通过学习了Spark官网上的一些内容，参考了许多博客和文章，也尝试进行了一些初级的Spark编程后写的关于Spark的简要的说明，希望能讲明白Spark这个框架的一些原理，提供一个基础的入门教程。  

![Spark logo](http://spark.apache.org/images/spark-logo-trademark.png)    

<!--more-->

Spark是一个用于分布式数据处理和并行计算的开源项目，最早由UC Berkeley 的AMP 实验室开发，现在已经交由Apache开源项目组管理。Spark目前变得非常流行，跟其高效性，通用性和易于编程性都有很大关系。Spark在机器学习，大数据处理和实时数据处理，以及分布式的应用场景中都能充分发挥作用。  

**Spark程序计算很快。** 根据[Spark主页](http://spark.apache.org/)上的描述，Spark程序比基于Memory的[Hadoop](http://hadoop.apache.org/)(一个分布式系统基础架构)的MapReduce要快100倍，比基于硬盘的Hadoop MapReduce 快10倍。Spark之所以有如此快的速度，是因为采用了很多高效的方案，如采用懒惰模式，基于内存进行操作，对数据进行多种方式的缓存等等。  

**Spark程序易于编写。** Spark 原生是由Scala编写，现在支持Java，Scala，Python和R四种语言。这四种语言可以覆盖较大的开发者范围，像R是数据处理专家的拿手语言，而Java是Hadoop的开发语言，而且由于Spark对Hadoop的一定程度的兼容性，使得Hadoop开发者可以快速地转到Spark平台上来。而Python和Scala是现代化的编程语言，编程风格优雅，入门简单，所以开发者可以快速地开发出可以实际应用的程序。  

**Spark统一了本地和分布式情形下的数据访问模式**。在本地电脑上，Spark会开多个进程来模拟分布式环境下的任务计算，所以即使在单机环境下，开发者也可以编写适用于分布式环境的程序，这大大地简化了程序的调试难度，也进一步加快了项目的开发进程。另外，Spark提出了弹性分布式数据集（RDD， Resilient Distributed Dataset）的数据格式,这种格式的数据默认就是分布式分布地，但是操作方式却和本地操作方式一样，即替开发者完成了运算节点之间拷贝数据的操作，使得开发人员像编写本地程序一样来编写分布式程序，毫无疑问这是一个很大的优势。    

上面是一些比较大范围的说明，而我个人对Spark比较向往的地方则是相比Hadoop，Spark上手很容易，官网上提供的教程和说明非常详尽，自己写一个计算$\pi$的程序只需要以下几行Python代码即可完成（代码来自Spark官方给出的例子）：
```python
import sys
from random import random
from operator import add

from pyspark import SparkContext


if __name__ == "__main__":
    """
        Usage: pi [partitions]
    """
    sc = SparkContext(appName="PythonPi")
    partitions = int(sys.argv[1]) if len(sys.argv) > 1 else 2
    n = 100000 * partitions

    def f(_):
        x = random() * 2 - 1
        y = random() * 2 - 1
        return 1 if x ** 2 + y ** 2 < 1 else 0

    count = sc.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
    print("Pi is roughly %f" % (4.0 * count / n))

    sc.stop()
```

可以看到，核心代码不超过10行。    

而为了配置Hadoop，我花了2天的时间，也还没有搞好，实在是对入门者不够友好。此外Java编写的程序和XML编写的配置文件一开始就有一种很“重”的感觉，使人望而却步。   

上面这部分内容是关于Spark的一个大概的介绍，**下面，我将从核心概念，集群模型和编程体验这三个大的方向进行详细的说明和我的理解。注意：下面的示例都以Spark的Python API为例。**

##核心概念
##1. SparkContext
Spark是管理集群和协调集群进程的对象。SparkContext就像任务的分配和总调度师一样，处理数据分配，任务切分这些任务。下图是Spark官网给出的集群之间的逻辑框架图，可以看到SparkContext在Driver程序中运行，这里的Driver就是主进程的意思。Worker Node就是集群的计算节点，计算任务在它们上完成。  
![Spark集群逻辑框架](http://spark.apache.org/docs/latest/img/cluster-overview.png)

Spark提供了Scala和Python的交互式命令环境，里面默认会创建一个`SparkContext`变量，并将其重命名为`sc`，所以在交互式环境下，可以用`sc`来方便地调用`SparkContext`的函数集合。下面示例中采用`sc`来代表`SparkContext`。  

###2. RDD
RDD是Resilient Distributed Datasets的缩写，中文翻译为弹性分布式数据集，它是Spark的数据操作元素，是具有容错性的并行的基本单元。**RDD之于Spark，就相当于array之于Numpy，Matrix之于MatLab，DataFrames之于Pandas。** 很重要的一个点是：RDD天然就是在分布式机器上存储的，比如对于下面这个RDD数据,可能Data1-3是存储在节点1的，Data4-6是存储节点2的，后面的数据也是这样，存储在集群中不同的机器上的。这种碎片化的存储使得任务的并行变得容易。    

![RDD data](http://img.ptcms.csdn.net/article/201511/25/5655b0ea512a8.jpg)

  
RDD生成也很容易，可以由串行的List， Tuple等等来生成，如：
```python
data = [1,3,5,7,9]
dist_data = sc.parallelize(data)
```
这两行代码就可以将串行的数据转换为并行的RDD。  

另一种生成RDD的方法是从外部的存储系统进行引用，如可以从硬盘上的文件（像‘data.txt’）,HDFS文件系统，HBase数据库，或者任何的提供Hadoop的InputFormat格式的数据来源都可以。对于各种格式的数据，Spark都有专门的处理函数，像`textFile`用来读取硬盘上的文本文件，按行返回文本中的内容；而`newAPIHadoopRDD`函数则可以保存/读取符合Haddoop输出/输入格式的文件。具体使用规则请参考[Spark编程指南](http://spark.apache.org/docs/latest/programming-guide.html)。  

###3. Action v.s. Transformation
RDD支持2种操作，一种是`Transformation`，这种操作的结果是生成一个新的RDD对象，即由RDD生成RDD，如Transformation操作`map`，就是对RDD中的每个数据，对应生成map函数中定义的数据，最后得到的还是一个RDD。举个具体的例子，假设map函数是：对RDD中的每个数据加1，假设原先的数据是[1,3,5,7,9],则这个map函数作用的结果是[2,4,6,8,10],仍然是个RDD（注意：这里为了方便解释，将RDD写出Python中的List形式，实际上要记得这里的RDD数据是保存在不同机器上的）。另一种操作叫做`Action`，这种操作的结果是得到一个值(Value)。即由RDD得到Value。如Action操作`reduce`，假设reduce函数设定为：求RDD中所有元素的和，则对该RDD作用reduce的结果是30,为一个值。  

常见的`Tranformation`操作包括`map，filter，flatMap,mapPartions, mapPartitionsWithIndex, sample, union, intersection, distinct, groupByKey, reduceByKey, aggregateByKey, sortByKey, join, pipe`等。    
常见的`Action`操作包括`reduce，collect，count，first，take，takeSampke， takeOrdered， countByKey, foreach`等等。  

###4. Lazy Evalution
Spark采用了惰性计算。所谓惰性计算，即对所有`transformation`，不会立即执行，而是等到某个`action`作用的时候，需要向Driver发送结果的时候再执行之前的所有`transformation`。简单来说，就是所有任务都拖到不能再拖的时候再执行。  

惰性计算能提高Spark运行的性能。试想，如果对所有的`transformation`操作，立即计算，然后向Dirver返回结果，则需要发送数目巨大的数据集；而如果采用惰性计算，则只需发送最后的一个值给Driver，传输开销会大大地减小。  

**需要指出的是：在Spark中，所有`transformation`操作都采用惰性模式，而所有`action`都是非惰性模式。**

###5. Closure
在Spark中执行某一项任务的时候，Spark driver程序会将RDD的的操作分配到各个计算节点上，Spark称这些计算节点为`executor`。而每个executor执行计算的变量和操作就称为这个executor的`Closure`。  

需要注意的是，各个executor的closure是不同的，刚开始的时候数据都从driver程序中克隆过来，之后这些数据就和driver程序中的数据没有任何关系了。这里可以类比`fork`操作，子进程和父进程之间的数据是隔离的，互不影响的。  

**由于各个executor和driver的数据是不同的，所以涉及到不同节点上同名变量的运算，结果结果是不确定的，也不要依赖于该运算结果。**


###6. Shuffle
在Spark中，有的时候为了执行某一个操作，需要从多个节点获取数据到一个节点，然后进行计算。计算后将计算结果再传给相应的计算节点。这个过程中，计算前后对应节点的数据是对应的，即节点1的计算结果还是返回到节点1,但是返回的顺序可能发生了改变，如节点1原先顺序是[2,3,4],可能结果是按[3,2,4]的计算结果返回的，这样就间接地完成了一个打乱顺序的操作，在Spark中称以上这个过程为`shuffle`。

由上述描述可以看出来，Shuffle操作是一个开销比较大的操作，需要较大量的硬盘IO，数据串行化操作，和网络IO。此外，为了在单个节点保存多个节点上传过来的数据，还需要消耗较大的内存空间。  

此外，Spark内部会隐式地**在硬盘上**保存该过程中产生的中间文件，以便于以后再次使用。过一定时间后，或者数据不再使用时，垃圾回收机器（GC，Garbage Collection）就会删除这些文件。由于GC回收的时间间隔会比较长，所以在运行Spark的过程中会产生很多的中间数据，占据很多的硬盘空间，所以Spark快，是以占据大量内存空间和磁盘空间作为代价的。  


###7. Persistance
为了加快运行的速度，Spark提供了`persist`和`cache`函数由开发者来显式地缓存RDD数据。在初次执行某个`action`的时候，对RDD数据进行缓存，在以后的`action`操作中，直接读取缓存的RDD数据。这样下来，`action`的执行速度可以提升10倍。  
Spark的缓存具有容错性，如果一个节点的RDD数据部分丢失了，则Spark会根据生成该部分RDD数据的`transformation`重新生成完全一样的数据。  

此外，Spark还允许设置不同的缓存存储级别（`StorageLevel`），如只缓存在内存中（`MEMORY_ONLY`），缓存在内存和硬盘中（`MEMORY_AND_DISK`），等等。这些参数可以通过`persist`函数进行设置。而`cache`函数则是`persist`函数指定`StorageLevel`为`MEMORY_ONLY`时的简写。    

本质上StorageLevel的选取，是在内存占用量和CPU高效性之间的平衡。Spark官方文档中推荐使用`MEMORY_ONLY`，如果不行，可以选用`MEMROY_ONLY_SER`，这中方式类似于前者，只不过是串行存储以节省开销。一般不建议用`DISK`相关的存储。  

Spark会自动监控缓存数据的使用情况，如果空间不够的话，就会使用最近使用次数最少算法（LRU，Least-Recently -Used）将部分缓存数据给删除掉。如果你想手动删除缓存，可以调用`RDD.unpersist()`函数。  


###8. Shared Variables
通常情况下，当Driver程序给各个cluster节点分配后任务，复制完初始数据后，各个节点就在自己的本地空间上单独进行计算，再也不会和Driver程序之间发送数据了。但是为了几个非常常用的操作，Spark提供了2类共享变量：`broadcast variable`和`accumulator`。  

broadcast变量是一种只读的变量，在driver进程需要向多个机器发送相同数据的时候会用到。并且规定boroadcast变量在广播后不可以被改变。我们可以对变量`v`进行broadcast操作，对其进行广播，然后在各个机器上使用的时候，使用`.value`来读取，而不是直接读取`v`的值。如下例：
```python
broadcastVar = sc.broadcast([1, 2, 3])
broadcastVar.value 
#结果：[1, 2, 3]
```
可以看到原理跟MPI里面的`MPI_Broadcast`函数的原理是比较类似的。  

另一种共享变量是Accumulator，通过`SparkContext.accumulator(v)`函数初始化为`v`，然后可以通过将各个进程中的值增加到这个变量上面，然后计算得到相应的值。Spark内置了数值类型的Accumulator变量，开发者可以自己实现别的类型的Accumulator变量。其值也通过`value`属性来获得。下面是一个计算各个节点上数据之和的例子：
```python
accum = sc.accumulator(0)
sc.parallelize([1, 2, 3, 4]).foreach(lambda x: accum.add(x))
accum.value 
#结果：10
```

##集群模型
结束了冗长而且枯燥的概念部分后，下面我来阐述一下关于Spark集群模型的一些理解。  

###1. Cluster模型
![Spark Cluster Model](http://spark.apache.org/docs/latest/img/cluster-overview.png)  
上图是官网给出的Spark集群模型，Driver Program 是主进程，SparkContext运行在它上面，它跟Cluster Manager相连。Driver对Cluster Manager下达任务人，然后由Cluster Manager将任资源分配给各个计算节点(Worker Node)上的`executor`，然后Driver再将应用的代码发送给各个Worker Node。最后，Driver向各个节点发送`Task`来运行。  

这里有几个需要注意的点：
> * 在Spark中，各个应用之间数据是隔离的，即不同的SparkContext之间互不可见。这样能有效地保护数据的局部性。  
> * Cluster Manager对Driver来说是不知的，透明的，只要能满足要求就可以。所以Spark可以在Mesos和YARN这些Cluster Manager上运行。  
> * 在运行过程中，Driver需要随时准备好接收来自各个计算节点的数据，所以对各个executor来说，Driver必须是可寻址的，比如有公网IP，或者如果在同一个局域网的话，有固定的局域网IP。  
> * 由于Driver需要随时接收消息和数据，所以最好Driver和各个节点比较邻近，这样数据传输会比较快。  

###2. Cluster Manager 类型
当前Spark支持3种类型的Cluster Manager,分别是：
> + `Apache Mesos`： [Mesos](http://spark.apache.org/docs/latest/running-on-mesos.html)是一种通用的的集群管理系统，可以运行Hadoop和别的分布式计算。
> + `Hadoop YARN`: 这是Hadoop 2 默认的资源管理系统。
> + `Standalone `-这种类型是Spark单独设计的管理系统，比较简单，也没有太多的需要预先学习的东西。

###3. 概念辨析
Spark集群模型有许多概念，之间的区别还是需要仔细辨析才能搞清楚。下面是从官方网站上抄录下来的一个定义，因为怕翻译后改变原意所以这里没有翻译，仅给出原文供参考：  
> * `Application` : User program built on Spark. Consists of a driver program and executors on the cluster.  
> * `Application jar` : A jar containing the user's Spark application. In some cases users will want to create an "uber jar" containing their application along with its dependencies. The user's jar should never include Hadoop or Spark libraries, however, these will be added at runtime.  
> * `Driver program` : The process running the main() function of the application and creating the SparkContext  
> * `Cluster manager` : An external service for acquiring resources on the cluster (e.g. standalone manager, Mesos, YARN)  
> * `Deploy mode` : Distinguishes where the driver process runs. In "cluster" mode, the framework launches the driver inside of the cluster. In "client" mode, the submitter launches the driver outside of the cluster.
> * `Worker node` : Any node that can run application code in the cluster
> * `Executor` : A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them. Each application has its own executors.
> * `Task`: A unit of work that will be sent to one executor
> * `Job` : A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action (e.g. save, collect); you'll see this term used in the driver's logs.
> * `Stage` : Each job gets divided into smaller sets of tasks called stages that depend on each other (similar to the map and reduce stages in MapReduce); you'll see this term used in the driver's logs.

###4. 资源监控
Spark在运行过程中，会在Driver程序所在机器的4040端口显示关于运行任务，存储情况和工作节点等等的Web UI。对于Standalone模式，在7070端口有类似的信息展示。开发者可以通过访问这个Web UI来了解更多信息。  

集群模型就这些内容，下面以Python编程为例，展示Spark编程的风格和思路。  

##编程体验
在这部分，我以WordCount 和计算PI这2个程序作为例子，描述如何用Python进行Spark编程。
###1. 下载Spark程序
从[Spark官方下载页面](http://spark.apache.org/downloads.html)选择一个合适版本的Spark。建议在`package type`这一栏选择`Pre-built for Hadoop 2.x and later`，这样下载下来的版本会自带Hadoop相关的东西，不用自己单独再配Hadoop。  
下载下来后，解压即可：
```bash
tar -xvf spark-*.tgz
```

###2. 打开Python命令行
进入解压后的目录，输入`./bin/pyspark`即可打开Python交互式窗口。这里会采用系统默认的Python交互式界面，如果想用体验更好的IPython交互式界面，则可以在输入命令之前设置如下环境变量：
```bash
export PYSPARK_DRIVER_PYTHON=ipython ./bin/pyspark
```
然后输入`./bin/pyspark`即可进入IPython。  
前面也提到过，在命令行下，SparkContext会自动创建好，并重命名为sc，所以下面可以直接使用sc来进行操作。  

###3. 读取Spark根目录下`REAMDE.md`中出现`Spark`这个单词的行数
为了完成这个任务，我们首先读取`README.md`作为RDD数据。还记得RDD吗？这是Spark默认的处理类型，默认就是分布式存储的。读取本地文本文件使用`textFile`函数。
```python
readMeFile = sc.textFile('README.md')
```
读进来的文件存在readMeFile这个RDD类型数据中，按行存储，其中每行就是`README.md`文件中的一行。  
然后可以使用`filter`操作来获取满足条件的数据：
```python
linesWithSpark = readMeFile.filter(lambda line : "Spark" in line)
```
这里`filter`函数返回满足里面lambda函数的新的RDD数据。lambda函数是Python中一种单行的函数，以一个语句来实现一个函数的功能。lambda后面紧跟的那个引号之前的变量为输入参数，引号后面的内容为输出结果，如：
```python
lambda x, y : x + y
```
就是返回x和y之和的一个lambda函数。  
要注意的是得到的RDD虽然是只包含字符串"Spark"的那些行，但还是分布式存储的。为了得到具体的行数，我们可以采用`count`函数：
```python
linesWithSpark.count()
#结果：15
```
此外，我们还可以把以上所有的`transformation`操作都以链式方式写在一起，如下：
```python
readMeFile.filter(lambda line : "Spark" in line).count()
```
如果将上述代码写成单独的可执行的Python文件，内容将会是：
```python
from pyspark import SparkContext

sc = SparkContext(appName="WordCount")
readMeFile = sc.textFile('README.md')
readMeFile.filter(lambda line : 'Spark' in line).count()
```

可以看到，很简单吧，下面我们继续来看用Spark来计算Pi值的例子。  

###4. 用Spark计算Pi（采用随机投点法）
所谓随机投点法，是根据圆和其外接正方形的面积之比为PI/4，因此我们可以统计在这个单位正方形内随机投点时，落入圆的比例为多少，投点数量足够多时，这个比例近似为PI/4,然后这个比例*4即为PI值。实际投点时，采取第一象限的[0,1]x[0,1]区域即可。  
首先我们定义一个函数`f`,这个函数进行每次随机投点的统计，是否落在圆内，落在圆内返回1,否则返回0：
```python
from random import random
def f(_):
    x = random() * 2 - 1
    y = random() * 2 - 1
    return 1 if x ** 2 + y ** 2 < 1 else 0
```
之后，我们共进行10^6次试验，每次试验调用f函数，然后把所有结果相加，最后再*4/10^6即为PI的估计。
```python
n = 10**6
count = sc.parallelize(range(1,n+1)).map(f).reduce(lambda x, y : x + y)
pi = 4.0 * count / n
print('*****result: pi is :%f*****' %(pi))  
```
其中第2行为主要的计算任务，搞懂这一行的操作大概就能明白Spark是怎么工作的了。  
将上述代码完成写出来，如下：
```python
#file name: calc_pi.py
from pyspark import SparkContext
from random import random

sc = SparkContext(appName='CalcPi')

def f(_):
    x = random() * 2 - 1
    y = random() * 2 - 1
    return 1 if x ** 2 + y ** 2 < 1 else 0

n = 10 ** 6
count = sc.parallelize(range(1, n + 1)).map(f).reduce(lambda x, y : x + y)
pi = 4.0 * count / n

print('*****result: pi is :%f*****' %(pi))  
```
可以看到，内容很简洁，比MPI复杂的函数命名简洁多了。  
之后，在Spark根目录中，使用如下命令开始运行Spark进行计算：
```bash
./bin/spark-submit calc_pi.py
```
可以看到会输出很多`INFO` 开头的信息，这里我将所有的输出都写下来，虽然内容很多，有些没有必要看，但我觉得如果仔细看这些输出的话，很能增加对Spark的理解，所以这里我还是不厌其烦地把所有输出信息都列出来了。  
```
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
16/05/16 21:33:56 INFO SparkContext: Running Spark version 1.6.1
16/05/16 21:33:56 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
16/05/16 21:33:56 WARN Utils: Your hostname, ustc resolves to a loopback address: 127.0.1.1; using 192.168.102.77 instead (on interface eth0)
16/05/16 21:33:56 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
16/05/16 21:33:56 INFO SecurityManager: Changing view acls to: yunfeng
16/05/16 21:33:56 INFO SecurityManager: Changing modify acls to: yunfeng
16/05/16 21:33:56 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(yunfeng); u
sers with modify permissions: Set(yunfeng)
16/05/16 21:33:56 INFO Utils: Successfully started service 'sparkDriver' on port 53174.
16/05/16 21:33:56 INFO Slf4jLogger: Slf4jLogger started
16/05/16 21:33:56 INFO Remoting: Starting remoting
16/05/16 21:33:56 INFO Remoting: Remoting started; listening on addresses :[akka.tcp://sparkDriverActorSystem@192.168.102.77:57025]
16/05/16 21:33:56 INFO Utils: Successfully started service 'sparkDriverActorSystem' on port 57025.
16/05/16 21:33:56 INFO SparkEnv: Registering MapOutputTracker
16/05/16 21:33:57 INFO SparkEnv: Registering BlockManagerMaster
16/05/16 21:33:57 INFO DiskBlockManager: Created local directory at /tmp/blockmgr-2ace648a-937b-4a4c-b984-6e4cd06b8273
16/05/16 21:33:57 INFO MemoryStore: MemoryStore started with capacity 511.5 MB
16/05/16 21:33:57 INFO SparkEnv: Registering OutputCommitCoordinator
16/05/16 21:33:57 INFO Utils: Successfully started service 'SparkUI' on port 4040.
16/05/16 21:33:57 INFO SparkUI: Started SparkUI at http://192.168.102.77:4040
16/05/16 21:33:57 INFO Utils: Copying /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py to /tmp/spark-6cb08b18-143f-42dc-88c3-2778646
0836b/userFiles-ae0a9fc0-65cf-467e-848b-4f3cf4e6e1c2/calc_pi.py
16/05/16 21:33:57 INFO SparkContext: Added file file:/home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py at file:/home/yunfeng/Download
s/spark-1.6.1-bin-hadoop2.6/calc_pi.py with timestamp 1463405637243
16/05/16 21:33:57 INFO Executor: Starting executor ID driver on host localhost
16/05/16 21:33:57 INFO Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 43770.
16/05/16 21:33:57 INFO NettyBlockTransferService: Server created on 43770
16/05/16 21:33:57 INFO BlockManagerMaster: Trying to register BlockManager
16/05/16 21:33:57 INFO BlockManagerMasterEndpoint: Registering block manager localhost:43770 with 511.5 MB RAM, BlockManagerId(driver, localhost
, 43770)
16/05/16 21:33:57 INFO BlockManagerMaster: Registered BlockManager
16/05/16 21:33:57 INFO SparkContext: Starting job: reduce at /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py:12
16/05/16 21:33:57 INFO DAGScheduler: Got job 0 (reduce at /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py:12) with 8 output partiti
ons
16/05/16 21:33:57 INFO DAGScheduler: Final stage: ResultStage 0 (reduce at /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py:12)
16/05/16 21:33:57 INFO DAGScheduler: Parents of final stage: List()
16/05/16 21:33:57 INFO DAGScheduler: Missing parents: List()
16/05/16 21:33:57 INFO DAGScheduler: Submitting ResultStage 0 (PythonRDD[1] at reduce at /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_
pi.py:12), which has no missing parents
16/05/16 21:33:57 INFO MemoryStore: Block broadcast_0 stored as values in memory (estimated size 4.3 KB, free 4.3 KB)
16/05/16 21:33:57 INFO MemoryStore: Block broadcast_0_piece0 stored as bytes in memory (estimated size 2.8 KB, free 7.1 KB)
16/05/16 21:33:57 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on localhost:43770 (size: 2.8 KB, free: 511.5 MB)
16/05/16 21:33:57 INFO SparkContext: Created broadcast 0 from broadcast at DAGScheduler.scala:1006
16/05/16 21:33:57 INFO DAGScheduler: Submitting 8 missing tasks from ResultStage 0 (PythonRDD[1] at reduce at /home/yunfeng/Downloads/spark-1.6.
1-bin-hadoop2.6/calc_pi.py:12)
16/05/16 21:33:57 INFO TaskSchedulerImpl: Adding task set 0.0 with 8 tasks
16/05/16 21:33:57 WARN TaskSetManager: Stage 0 contains a task of very large size (486 KB). The maximum recommended task size is 100 KB.
16/05/16 21:33:57 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, localhost, partition 0,PROCESS_LOCAL, 497894 bytes)
16/05/16 21:33:57 INFO TaskSetManager: Starting task 1.0 in stage 0.0 (TID 1, localhost, partition 1,PROCESS_LOCAL, 629219 bytes)
16/05/16 21:33:57 INFO TaskSetManager: Starting task 2.0 in stage 0.0 (TID 2, localhost, partition 2,PROCESS_LOCAL, 629219 bytes)
16/05/16 21:33:57 INFO TaskSetManager: Starting task 3.0 in stage 0.0 (TID 3, localhost, partition 3,PROCESS_LOCAL, 629219 bytes)
16/05/16 21:33:57 INFO TaskSetManager: Starting task 4.0 in stage 0.0 (TID 4, localhost, partition 4,PROCESS_LOCAL, 629219 bytes)
16/05/16 21:33:57 INFO TaskSetManager: Starting task 5.0 in stage 0.0 (TID 5, localhost, partition 5,PROCESS_LOCAL, 629219 bytes)
16/05/16 21:33:57 INFO TaskSetManager: Starting task 6.0 in stage 0.0 (TID 6, localhost, partition 6,PROCESS_LOCAL, 629219 bytes)
16/05/16 21:33:57 INFO TaskSetManager: Starting task 7.0 in stage 0.0 (TID 7, localhost, partition 7,PROCESS_LOCAL, 632117 bytes)
16/05/16 21:33:57 INFO Executor: Running task 3.0 in stage 0.0 (TID 3)
16/05/16 21:33:57 INFO Executor: Running task 1.0 in stage 0.0 (TID 1)
16/05/16 21:33:57 INFO Executor: Running task 0.0 in stage 0.0 (TID 0)
16/05/16 21:33:57 INFO Executor: Running task 6.0 in stage 0.0 (TID 6)
16/05/16 21:33:57 INFO Executor: Running task 7.0 in stage 0.0 (TID 7)
16/05/16 21:33:57 INFO Executor: Running task 2.0 in stage 0.0 (TID 2)
16/05/16 21:33:57 INFO Executor: Running task 5.0 in stage 0.0 (TID 5)
16/05/16 21:33:57 INFO Executor: Running task 4.0 in stage 0.0 (TID 4)
16/05/16 21:33:57 INFO Executor: Fetching file:/home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py with timestamp 1463405637243
16/05/16 21:33:57 INFO Utils: /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py has been previously copied to /tmp/spark-6cb08b18-143
f-42dc-88c3-27786460836b/userFiles-ae0a9fc0-65cf-467e-848b-4f3cf4e6e1c2/calc_pi.py
16/05/16 21:33:58 INFO PythonRunner: Times: total = 340, boot = 226, init = 1, finish = 113
16/05/16 21:33:58 INFO Executor: Finished task 7.0 in stage 0.0 (TID 7). 998 bytes result sent to driver
16/05/16 21:33:58 INFO PythonRunner: Times: total = 353, boot = 222, init = 2, finish = 129
16/05/16 21:33:58 INFO PythonRunner: Times: total = 359, boot = 230, init = 1, finish = 128
16/05/16 21:33:58 INFO PythonRunner: Times: total = 360, boot = 225, init = 3, finish = 132
16/05/16 21:33:58 INFO PythonRunner: Times: total = 358, boot = 224, init = 1, finish = 133
16/05/16 21:33:58 INFO Executor: Finished task 5.0 in stage 0.0 (TID 5). 998 bytes result sent to driver
16/05/16 21:33:58 INFO Executor: Finished task 4.0 in stage 0.0 (TID 4). 998 bytes result sent to driver
16/05/16 21:33:58 INFO Executor: Finished task 0.0 in stage 0.0 (TID 0). 998 bytes result sent to driver
16/05/16 21:33:58 INFO PythonRunner: Times: total = 373, boot = 248, init = 0, finish = 125
16/05/16 21:33:58 INFO Executor: Finished task 1.0 in stage 0.0 (TID 1). 998 bytes result sent to driver
16/05/16 21:33:58 INFO Executor: Finished task 2.0 in stage 0.0 (TID 2). 998 bytes result sent to driver
16/05/16 21:33:58 INFO TaskSetManager: Finished task 7.0 in stage 0.0 (TID 7) in 420 ms on localhost (1/8)
16/05/16 21:33:58 INFO TaskSetManager: Finished task 5.0 in stage 0.0 (TID 5) in 427 ms on localhost (2/8)
16/05/16 21:33:58 INFO PythonRunner: Times: total = 385, boot = 245, init = 0, finish = 140
16/05/16 21:33:58 INFO Executor: Finished task 6.0 in stage 0.0 (TID 6). 998 bytes result sent to driver
16/05/16 21:33:58 INFO TaskSetManager: Finished task 4.0 in stage 0.0 (TID 4) in 431 ms on localhost (3/8)
16/05/16 21:33:58 INFO TaskSetManager: Finished task 1.0 in stage 0.0 (TID 1) in 439 ms on localhost (4/8)
16/05/16 21:33:58 INFO TaskSetManager: Finished task 2.0 in stage 0.0 (TID 2) in 437 ms on localhost (5/8)
16/05/16 21:33:58 INFO TaskSetManager: Finished task 6.0 in stage 0.0 (TID 6) in 430 ms on localhost (6/8)
16/05/16 21:33:58 INFO TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 455 ms on localhost (7/8)
16/05/16 21:33:58 INFO PythonRunner: Times: total = 390, boot = 246, init = 1, finish = 143
16/05/16 21:33:58 INFO Executor: Finished task 3.0 in stage 0.0 (TID 3). 998 bytes result sent to driver
16/05/16 21:33:58 INFO TaskSetManager: Finished task 3.0 in stage 0.0 (TID 3) in 442 ms on localhost (8/8)
16/05/16 21:33:58 INFO DAGScheduler: ResultStage 0 (reduce at /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py:12) finished in 0.467
 s
16/05/16 21:33:58 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool 
16/05/16 21:33:58 INFO DAGScheduler: Job 0 finished: reduce at /home/yunfeng/Downloads/spark-1.6.1-bin-hadoop2.6/calc_pi.py:12, took 0.569039 s
*****result:pi is :3.140324*****
16/05/16 21:33:58 INFO SparkContext: Invoking stop() from shutdown hook
16/05/16 21:33:58 INFO SparkUI: Stopped Spark web UI at http://192.168.102.77:4040
16/05/16 21:33:58 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
16/05/16 21:33:58 INFO MemoryStore: MemoryStore cleared
16/05/16 21:33:58 INFO BlockManager: BlockManager stopped
16/05/16 21:33:58 INFO BlockManagerMaster: BlockManagerMaster stopped
16/05/16 21:33:58 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
16/05/16 21:33:58 INFO SparkContext: Successfully stopped SparkContext
16/05/16 21:33:58 INFO ShutdownHookManager: Shutdown hook called
16/05/16 21:33:58 INFO ShutdownHookManager: Deleting directory /tmp/spark-6cb08b18-143f-42dc-88c3-27786460836b/pyspark-33d22309-ef12-45d6-9862-2
5ceb8beadac
16/05/16 21:33:58 INFO ShutdownHookManager: Deleting directory /tmp/spark-6cb08b18-143f-42dc-88c3-27786460836b
16/05/16 21:33:58 INFO RemoteActorRefProvider$RemotingTerminator: Shutting down remote daemon.
```

可以看到在96行，输出了我们想要的结果：
```bash
*****result:pi is :3.140324*****
```
需要注意的是：Spark自动在本地开了8个进程，来模拟在分布式情况下的计算节点，这样就可以在单机情况下调试适用于分布式情况下的代码了。  

###5. 在分布式环境下部署
在单机上调试好程序后，我们就可以将代码部署到分布式的机器上了。**这里有个要求：每个分布式的机器节点上都必须安装相同版本的Spark。**所以第一步就是再各个机器上安装Spark。  

安装完Spark后，我们就可以通过下面的命令来启动各个节点的Spark了：  
1.在要运行Driver程序（master）的机器上，在Spark根目录下，执行命令：
```bash
 ./sbin/start-master.sh
```
2.在**各个Worker Node上**，连接到主节点上：
```bash
./sbin/star-slave.sh <master-spark-URL>
```

这是一种手动启动的方式。此外，还可以通过在Driver 程序所在节点上执行下面的命令来自动地启动或停止所有节点的Spark程序：
* `sbin/start-master.sh` ： 启动主进程 
* `sbin/start-slaves.sh` ： 启动`conf/slaves`文件里面的所有节点
* `sbin/start-all.sh` ： 启动主进程和所有计算节点
* `sbin/stop-master.sh`： 停止主进程
* `sbin/stop-slavers.sh` ： 停止所有计算节点

配置完分布式环境后，就可以运行程序了。以上述的`calc_pi.py`为例，假设master程序运行在192.168.3.2:8080，则在运行master的主机上运行如下命令：
```bash
./bin/spark-submit -master spark://192.168.3.2:8080 calc_pi.py
```
这样就可以分布式地运行Spark了！

至此Spark的内容的总结就结束了，总的来说，Spark编程并没有想象中的那么复杂，恰恰相反，随着时间的推移，这些开发工具越来越对开发者友好，这也是使得Spark能轻易地上手的原因。  

