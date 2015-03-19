## shell

### 1. variable 
```shell
echo "hello world, please input"
while read var
do
        echo "hello, $var"
        break
done
```

### 2. if
```shell
echo "please cin your score : "
while read scores;do
        if [[ $scores -gt 90 ]]; then
                echo "very good";
        elif [[ $scores -gt 80 ]]; then
                echo "good";
        elif [[ $scores -gt 60 ]]; then
                echo "pass";
        else
                echo "no pass";
        fi;
        echo "please cin your score : "
done
```

### 3. while
```shell
i=10;
while [[ $i -gt 5 ]];do
        echo $i;
        ((i--));
done;
```

### 4. until
```shell
i=10;
until [[ $i -lt 0 ]];do
        echo $i;
        ((i--));
done;
```

### 5. for
```shell
for i in $(seq 10); do
        echo $i
done


for((i=0;i<5;i++));do
        echo $i;
done;
```

### 6. date
```shell
tomorrow=`date -d "+1 day" +%Y-%m-%d`
echo $tomorrow
```

### 7. select
```shell
select ch in "begin" "end" "exit"
do
case $ch in
"begin")
        echo "start sth"
        ;;
"end")
        echo "stop sth"
        break;
        ;;
"exit")
        echo "exit"
        break;
        ;;
*)
        echo "ignore"
        ;;
esac
done;
```

### 8. function
```shell
function printit() {
        echo -n "your choice is $1 "
}

echo "this program will print your selection !"
case $1 in
        "one")
                printit $1; echo $1 | tr 'a-z' 'A-Z'
                ;;
        "two")
                printit $1; echo $1 | tr 'a-z' 'A-Z'
                ;;
        "three")
                printit $1; echo $1 | tr 'a-z' 'A-Z'
                ;;
        *)
                echo "useage $0 {one|two threee}"
                ;;
esac
```

### 9. read file
```shell
#读取文件并输出
while read line;do
        echo $line;
done < /etc/hosts
-----------------------------------------------
#读取文件，按tab键进行切分
while read line;do
        echo $line;
        arr=(${line//\t/ })
        for i in ${arr[@]}
        do
                echo $i
        done;
done < test
-----------------------------------------------
#先对文件进行排序，然后判断输出，删除临时文件
sort -n -k 1 -t \t test -o outfile

while read line;do
        #echo $line;
        if [[ "$line" != "" ]];then
                arr=(${line//\t/ })

                if [[ ${arr[1]} -gt 0 || ${arr[1]} -lt 0 ]];then
                        echo "${arr[0]} ${arr[2]}       ${arr[4]}"
                #else
                        #echo "not a number"
                fi
        #else
                #echo "line has no content"
        fi
done < outfile
rm outfile
```
#### 注：sort用法
>* -b：忽略每行前面开始处的空格字符；>* -c：检查文件是否已经按照顺序排序，排序过为真；>* -d：排序时，处理英文字母、数字和空格字符，以字典顺序排序。忽略其他所有字符；>* -f：排序时，将小写字母视为大写字母；>* -i：排序时，处理040~176之间的ASCII字符，忽略其他所有字符；>* -m：将几个排序好的文件进行合并；>* -M：将前面3个字母按月份的缩写进行排序；>* -n：按照数值大小进行排序；>* -o outfile.txt：将排序后的结果存入outfile.txt；>* -r：以相反的顺序进行排序；>* -k：指定需要排序的列数（栏数）；>* -t 分隔符：指定排序时所用到的栏位分隔符；### 10. operate file
```shell
CURRENT=$(pwd)
USER=$(whoami)
for FILE in `find -user $USER -mtime -0.5 -type f`
do
        echo $FILE
done
```
#### 注：find：[详见](http://www.lampweb.org/linux/2/13.html)

### 11. crontab定时任务
[详见](https://github.com/me115/linuxtools_rst/blob/master/tool/crontab.rst)