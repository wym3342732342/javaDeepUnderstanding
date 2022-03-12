# Java Modularity

&emsp;&emsp;从JDK9开始，module成为与class，interface，package等同等重要的一等公民，是(需要成为)Javaers日常高频处理的词汇。Java的modularity起源于2008年的Jigsaw项目，并从2014年开始在JDK9的开发过程中设计并实现。引入module不像引入像lambada表达式仅是语法的改变，它涉及到了JLS（java language specification），JVM，JDK，JAR, JNI, JVM TI（tool interface）和JDWP(javadebug wire protocol，java调试通信协议)等多个模块的改动。本文结合一个样例从JDK引入module机制的原因，引入module涉及的改动点，module目标，module与jar的关系，module与reflection的关系，automatic module和unnamed module等方面介绍modularity相关机制，希望给想在实际项目中使用java module的程序员提供参考。

