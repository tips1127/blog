1. libs新建文件夹存放依赖所有jar包

2. cmd 执行：

2.1
```
javac -encoding UTF-8 -classpath .;D:\libs\commons-codec-1.12.jar;D:\libs\commons-lang3-3.9.jar -d . D:\libs\src\test\Test.java
```
```
1. encoding 防止中文乱码
2. -classpath   指定class文件路径
3. -d . 会在当前执行目录下生成 一个java所在的包名的（这里我的是test）目录，以及编译后的class文件
4. 指定要被编译的java文件
```

3.优化，由于依赖包很多，不可能每个都敲上去吧：

path_jars 可以是相对路径也可以是绝对路径
```
java -Djava.ext.dirs=path_jars package.className

java -Djava.ext.dirs=D:\libs package.className


java -Djava.ext.dirs=./ package.className


javac -encoding UTF-8 -Djava.ext.dirs=./ -d . C:\Users\chenquan\IdeaProjects\Toy\src\test\SignNatureTest.java

javac -encoding UTF-8 -Djava.ext.dirs=./ C:\Users\chenquan\IdeaProjects\Toy\src\test\SignNatureTest.java
```
JAVA 反编译(idea)
```
java -cp java-decompiler.jar org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler -dgs=true c:\my.jar d:\decompiled\
```