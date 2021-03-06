RHadoop实验 – 统计邮箱出现次数
==============

[RHadoop实践系列文章](http://blog.fens.me/series-rhadoop/)，包含了R语言与Hadoop结合进行海量数据分析。Hadoop主要用来存储海量数据，R语言完成MapReduce 算法，用来替代Java的MapReduce实现。有了RHadoop可以让广大的R语言爱好者，有更强大的工具处理大数据1G, 10G, 100G, TB, PB。 由于大数据所带来的单机性能问题，可能会一去不复返了。

RHadoop实践是一套系列文章，主要包括[”Hadoop环境搭建”](http://blog.fens.me/rhadoop-hadoop/)，[”RHadoop安装与使用”](http://blog.fens.me/rhadoop-rhadoop/)，[”R实现MapReduce的协同过滤算法”](http://blog.fens.me/rhadoop-mapreduce-rmr/)，[”HBase和rhbase的安装与使用”](http://blog.fens.me/rhadoop-hbase-rhase/)。对于单独的R语言爱好者，Java爱好者，或者Hadoop爱好者来说，同时具备三种语言知识并不容 易。此文虽为入门文章，但R,Java,Hadoop基础知识还是需要大家提前掌握。

#### 关于作者

+ 张丹(Conan), 程序员Java,R,PHP,Javascript
+ weibo：@Conan_Z
+ blog: http://blog.fens.me
+ email: bsspirit@gmail.com

#### 转载请注明出处：http://blog.fens.me/rhadoop-demo-email/

![RHadoop实验 – 统计邮箱出现次数](http://blog.fens.me/wp-content/uploads/2013/05/rhadoop-demo-email.png)

#### 目录
1. 需求描述
2. 实验数据
3. 算法实现

## 1.需求描述

基于RHADOOP通过rmr包实现MapReduce算法：

+ 1). 计算邮箱域出现了多少次  
+ 2). 按次数从大到小排序

例如：

163.com,14  
sohu.com,2

## 2.实验数据：

```{bash}
wolys@21cn.com
zss1984@126.com
294522652@qq.com
simulateboy@163.com
zhoushigang_123@163.com
sirenxing424@126.com
lixinyu23@qq.com
chenlei1201@gmail.com
370433835@qq.com
cxx0409@126.com
viv093@sina.com
q62148830@163.com
65993266@qq.com
summeredison@sohu.com
zhangbao-autumn@163.com
diduo_007@yahoo.com.cn
fxh852@163.com
weiyang1128@163.com
licaijun007@163.com
junhongshouji@126.com
wuxiaohong11111@163.com
fennal@sina.com
li_dao888@163.com
bokil.xu@163.com
362212053@qq.com
youloveyingying@yahoo.cn
boiny@126.com
linlixian200606@126.com
alex126126@126.com
654468252@qq.com
huangdaqiao@yahoo.com.cn
kitty12502@163.com
xl200811@sohu.com
ysjd8@163.com
851627938@qq.com
wubo_1225@163.com
kangtezc@163.com
xiao2018@126.com
121641873@qq.com
296489419@qq.com
beibeilong012@126.com
```

## 3. 算法实现

### 1).计算邮箱域出现了多少次

```{bash}
library(rmr2)
data<-read.table(file="hadoop15.txt")
d0<-to.dfs(keyval(1, data))
from.dfs(d0)
```

输出：

```{bash}
$key
[1] 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
[39] 1 1 1
$val
V1
1 wolys@21cn.com
2 zss1984@126.com
3 294522652@qq.com
4 simulateboy@163.com
5 zhoushigang_123@163.com
6 sirenxing424@126.com
7 lixinyu23@qq.com
8 chenlei1201@gmail.com
9 370433835@qq.com
10 cxx0409@126.com
11 viv093@sina.com
12 q62148830@163.com
13 65993266@qq.com
14 summeredison@sohu.com
15 zhangbao-autumn@163.com
16 diduo_007@yahoo.com.cn
17 fxh852@163.com
18 weiyang1128@163.com
19 licaijun007@163.com
20 junhongshouji@126.com
21 wuxiaohong11111@163.com
22 fennal@sina.com
23 li_dao888@163.com
24 bokil.xu@163.com
25 362212053@qq.com
26 youloveyingying@yahoo.cn
27 boiny@126.com
28 linlixian200606@126.com
29 alex126126@126.com
30 654468252@qq.com
31 huangdaqiao@yahoo.com.cn
32 kitty12502@163.com
33 xl200811@sohu.com
34 ysjd8@163.com
35 851627938@qq.com
36 wubo_1225@163.com
37 kangtezc@163.com
38 xiao2018@126.com
39 121641873@qq.com
40 296489419@qq.com
41 beibeilong012@126.com

```

计算邮箱域出现了多少次
```{bash}
mr<-function(input=d0){
map<-function(k,v){
keyval(word(as.character(v$V1), 2, sep = fixed('@')),1)
}
reduce =function(k, v ) {
keyval(k, sum(v))
}
d1<-mapreduce(input=input,map=map,reduce=reduce,combine=TRUE)
}
d1<-mr(d0)
from.dfs(d1)
```

输出：
```{bash}
$key
[1] "126.com" "163.com" "21cn.com" "gmail.com" "qq.com"
[6] "sina.com" "sohu.com" "yahoo.cn" "yahoo.com.cn"
$val
[1] 9 14 1 1 9 2 2 1 2
```

### 2). 按次数从大到小排序

```{bash}
sort<-function(input=d1){
map<-function(k,v){
keyval(1,data.frame(k,v))
}
reduce<-function(k,v){
v2<-v[order(as.integer(v$v),decreasing=TRUE),]
keyval(1,v2)
}
d2<-mapreduce(input=input,map=map,reduce=reduce,combine=TRUE)
}
d2<-sort(d1)
result<-from.dfs(d2)
result$val
```

输出：
```{bash}
k v
2 163.com 14
1 126.com 9
5 qq.com 9
6 sina.com 2
7 sohu.com 2
9 yahoo.com.cn 2
3 21cn.com 1
4 gmail.com 1
8 yahoo.cn 1
```

#### 转载请注明出处：http://blog.fens.me/rhadoop-demo-email/
