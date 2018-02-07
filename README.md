#MYSQL
###　six

####　慢查询分析

1. 程序检索了超过需要的数据
    1.  查询不需要的记录：加上limit限制
    2.  多表关联返回全部列：取需要的列
    3.  总是取全部列
    4.  重复查询相同的数据:如取用户头像地址 
2. mysql是否在分析大量超过需要的数据
    1.  响应时间：分为服务时间和排队时间；
    2.  扫描的行数和返回的行数:多表查询中服务器需要扫描多行生成结果集中成一行。
    3.  扫描的行数和访问类型：
     - 全表扫描《索引扫描《范围扫描《唯一索引扫描《常数引用
     - 索引中使用where》索引覆盖扫描（using index）返回记录然后过滤》数据表返回所有数据（using where） 中过滤          
     - 相应优化：使用索引覆盖扫描，改变表结构（汇总表），重构查询。
***
####　重构查询方式       
1.  切分查询：将大查询分割为小查询。如删除数据，一次删除会锁住很多数据。
    ````
    do{
        row_affect=DELETE FROM messages WHERE create<DATE_SUB(NOW(),INTERVAL 3 MONTH)
        LIMIT 10000;
       }while row_affect>0
    ````
2.  分解查询：对每一个表进行一次单表查询，然后在应用中组合。
    -   缓存效率更高：应用或mysql缓存了某个单表查询数据，减少查询。
    -   减少锁的竞争。
    -   应用层做关联，更容易扩展。
    -   查询本身效率提升。
    -   减少冗余记录查询。
    -   相当于哈希关联
***
####　查询执行基础
1. 查询过程：
    -   客服端发送查询给mysql服务器
    -   服务器检查查询缓存，无下一步
    -   服务端进行sql解析、预处理、查询优化器生成查询执行计划
    -   mysql根据计划调用引擎查询数据
    -   返回结果
2.  客户端－服务端协议
    -   半双工协议，一方必须等待另一方传送完所有数据包    
    -   服务器会接收全部结果然后缓存（早释放查询），但当大数据时，库函数会花费大量时间和内存存储结果集（查询占用）
     
3.  查询状态
    -   Sleep:线程等待客服端发送新的请求    
    -   Query:线程正在执行查询或正在将结果发送给客户端
    -   Locked:线程等待表锁
    -   Analyzing and statistics：线程收集存储引擎统计信息，并生成查询计划
    -   Copying to tmp table：执行查询，并将结果复制到临时表（group by、排序）
    -   Sorting result：结果排序
    -   Sending data：多个状态间传送数据｜生成结果集|向客服端法送
4.  查询优化处理
    1.  语法解析器和预处理：语法解析器解析SQL语句，生成解析树，检查语句是否正确。预处理进一步检查解析树是否合法，列是否存在，名字是否有歧义，验证权限
    2.  查询优化器：动静优化
    3.  关联表优化：根据成本重构关联顺序
    4.  排序优化：成本高。小的内存排，大的分块内存排，存到磁盘中最后合并。
****
####优化器的局限性

1.  关联子查询:exists先查外表，in查内表，根据表大小选择。连接通常好于子查询
2.  union限制:限制条件无法推到内层，需要额外写。
3.  不支持松散索引：组合索引中只使用第二个，索引无效。（可以给前面列加常数值）
4.  最大值最小值优化：
    ````
       select MIN(id) from user where username='yy';
            改写
       select id from user use index(primary) where username='yy' limit 1;
    ````
5.  同一个表上查询更新：解决办法，通过多表关update
***
####查询优化提示


         