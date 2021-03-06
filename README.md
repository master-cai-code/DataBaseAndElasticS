# DataBase And ElasticSearch
## 各种树的对比描述
### 1.1 二叉搜索树：   根节点比左节点大，比右节点小
        运用了“二分法”思想 —————— “前小后大”.
        根节点比左节点大，比右节点小.
    缺点：会由于节点单调有序，退化成链表，丧失了O(logN)的高效-->O(N)
    
### 1.2 平衡二叉树：   解决二叉搜索树的缺点，不会退化成链表，保持左右子树的高度差的绝对值不大于1
     缺点：树太“瘦高”了，磁盘IO多，要变“矮胖”
     
### 1.3 B树：多阶平衡二叉树，一个节点有多个元素
        树的高度就是磁盘IO的次数，树越高-->磁盘IO越多-->查找时间越多  （CPU>>内存>>磁盘）
        一个节点里面元素间的比较是内存比较，时间远小于磁盘IO，可忽略不计.
        
### 1.4 B+树：改进B树
        B和B+的区别：
          1. B树中每个节点都保存记录，B+树只有叶子节点保存记录（并将叶子节点用链表连接）
          2. B+树的非叶节点没有卫星数据（索引指向的数据记录），每个非叶节点少了一个指针空间，可以存更多的索引，所以相同的索引，B+树又会更矮！
        总结B+树的优势：
             1.单一节点存储更多的元素，使得查询的IO次数更少；
             2.所有查询都要查找到叶子节点，查询性能稳定；
             3.所有叶子节点形成有序链表，便于范围查询。
         
         
### 1.5 红黑树：
               ......
               .....
               ..... 
               .......
               
               

## 索引：为增快查找效率，建立索引，将索引建成B+树，查找索引 （一个索引对应一个数据地址）
## Mysql的两种搜索引擎
  1. MyISAM  ：非聚集索引（索引和数据分开存储）  索引直接存的是数据地址
  2. InnoDB ： 聚集索引（索引和数据捆在一起）   索引下面就是数据
  
             InnoDB的数据文件本身要  按主键聚集  ，所以InnoDB要求表  必须有主键 （MyISAM可以没有），
             如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，
             如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。
             
             辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。
             （MyISAM都是直接找到索引对应地址的数据）
             
  ##  Lucene索引（也是ES基于的索引）
  ### 1.核心：倒排索引
         一行数据称为一个document，而且有一个docid；每个字段一个倒排索引，字段的值为term，以及term 出现的docid .
         
         docid  年龄  性别
           1     18    女
           2     20    女
           3     18    男

       这里每一行是一个document。每个document都有一个docid。那么给这些document建立的倒排索引就是：

       年龄
            18    [1,3]
            20    [2]
       性别 
            女    [1,2]
            男    [3]
         
     将所有的term排序后  组成了term dictionary，此时term dictionary太大了，只能放在磁盘上，进行磁盘IO，要想办法倒进内存；
     term index 由此产生！ （算索引的索引???）  
     
     term index 也是一棵树，保存term dictionary的部分前缀（可以有效压缩），保存在内存里面；
     根据term index 找到term dictionary中的 某个前缀 ，再从这个offset顺序向后查找（磁盘的顺序读写速度很快）
     找到具体的term后，进行一次随机读写磁盘IO （注意：只需要一次磁盘随机读写，速度快的原因所在！！！）
     
     总结：    多次内存比较 + 多次磁盘顺序比较 + 一次磁盘随机读写 ，前两者时间可忽略
