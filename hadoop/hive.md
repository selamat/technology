## hive

### 1. 创建表
```sql
### 创建user表，不同字段之间以‘\t’进行区分
create table user(ip string, name string, time string, url string) row format delimited fields terminated by '\t' stored as textfile

### -------------------------
### 创建user2表，按ds字段进行分区，不同字段之间以‘,’区分，并以文件形式存储
create table user2(ip STRING, name STRING, time STRING, url STRING) partitioned by (ds string) row format delimited fields terminated by ',' stored as textfile
```
#### 注：具体partition和bucket概念参加 [链接](http://my.oschina.net/leejun2005/blog/178631#OSC_h3_1)
### 2. 装入数据
```sql
### 将数据装入user2表，并放入2015-01-01这个分区中
load data local inpath './examples/files/user1.txt' overwrite into table user2 partition (ds='2015-01-01');
```

### 3. 查看分区
```sql
show partitions user2;
```
### 4. 查看全部表 and 查看表结构
```sql
show tables;
desc tablename;
```

### 5. 嵌入脚本执行
python 脚本

```python
import sys
import hashlib

for line in sys.stdin:
    line = line.strip()
    arr = line.split()
    md5_arr = []
    for a in arr:
        md5_arr.append(hashlib.md5(a).hexdigest())
    print "\t".join(md5_arr)
```
将python脚本加入hive中，并执行

```sql
hive> add file ./python/test.py;
hive> from user3
    > select transform(ip, name)
    > using 'python test.py'
    > as ip, name;
```
注：在执行python脚本时，创建表的字段分隔符需使用‘\t’

### 6. JAVA UDF
#### 创建maven工程，maven配置

```java
	<dependency>
		<groupId>org.apache.hadoop</groupId>
		<artifactId>hadoop-core</artifactId>
		<version>1.2.1</version>
	</dependency>
	
  	<dependency>  
        <groupId>org.apache.hadoop</groupId>  
        <artifactId>hadoop-common</artifactId>  
        <version>2.5.1</version>  
    </dependency>  
    <dependency>  
        <groupId>org.apache.hadoop</groupId>  
        <artifactId>hadoop-hdfs</artifactId>  
        <version>2.5.1</version>  
    </dependency>  
    <dependency>  
        <groupId>org.apache.hadoop</groupId>  
        <artifactId>hadoop-client</artifactId>  
        <version>2.5.1</version>  
    </dependency>
```
#### JAVA代码，Text输入（可定义其他参数），处理完毕后然会输出结果

```java
public class ToUpperCase extends UDF {  
	
    public Text evaluate(final Text s) {  
        if (s == null) 
        	return new Text("this text is null"); 
        return new Text(s.toString().toUpperCase());  
    }  
}  
```


#### 编写UDF函数并打包，在hive中添加jar包并创建临时函数

```sql
hive> add jar ./jar/com.ganji-0.0.1.jar;
hive> create temporary function touppercase as 'udf.com.ganji.ToUpperCase';
```

### 7. JAVA UDAF
#### maven配置同上
#### JAVA代码，其中迭代器为传入参数的类型（可设置多个），terminate函数为输出类型可参见[连接](http://blog.csdn.net/ruishenh/article/details/17577795)

```java
public class Mean2 extends UDAF {
	
	public static class AVGUFAFEvaluator implements UDAFEvaluator {
		
		public static class PartialResult {
			double sum;
			long count;
		}
		
		private PartialResult result = null;

		public void init() {
			result = null;
		}
		
		public boolean iterate(double value) { 
		    if (result == null)
		    	result = new PartialResult();  
		        
		    result.sum += value;  
		    result.count++;  
		    return true;  
		}
		
		public PartialResult terminatePartial() {  
	        return result;  
	    }  
		
		public boolean merge(PartialResult value) {  
	        if(value == null)
	        	return true;
	        
	        if (result == null) {  
	        	result = new PartialResult();  
	        }
	       
	        result.count += value.count;
	        result.sum += value.sum;
	        
	        return true;  
	    }  
		
		public double terminate() {  
			if(result == null)
				return 0.0;
			
			return result.sum / result.count;
		}
	}
}
```

#### 创建函数和使用见上
