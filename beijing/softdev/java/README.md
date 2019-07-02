这里记录关于软件开发的一些理解。

#### 知识体系分类
	1， Java，记录一些技术原理、架构、设计思想
	2， Linux，记录一些命令、脚本、思想

#### Java 类加载机制
    参考地址：https://www.liangzl.com/get-article-detail-19878.html
    从中也可以看出 GC 机制，实际上是制定一个class文件淘汰标准来保证系统资源的足够利用，只要满足这个标准的class，就会
    被JVM给卸载(unload)了,否则可能会发生内存溢出异常。MeteSpace: 元数据空间， 存在class文件