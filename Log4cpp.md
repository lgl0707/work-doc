**Log4cpp**



**windows****：环境** **win10 / VS2017**

1、下载与解压

 http://sourceforge.net/projects/log4cpp/  

解压

2、编译

打开msvc10\msvc10.sln；

只编译log4cpp和log4cppLIB    先log4cppLIB后log4cpp；不进行调试的话生成Release版

3、错误处理

​	1. error C2084: 函数“int snprintf(char *const,const std::size_t,const char *const,...)”已有主体

​	解决方法：
​	在log4cpp项目中找到snprintf.c文件，在编辑器中打开它，并定位到195行，找到有（/* #define HAVE_SNPRINTF 			*/）字样，去掉该位的注释/**/符号

2.  fatal error RC1110: could not open .\Debug\NTEventLogCategories.rc

   解决方法：
   将log4cppLIB工程中的NTEventLogCategories.mc文件删除，然后重新编译即可。

4、用的是log4cppLIB生成的库



问题：

debug的时候会报错

解决方法：看下日志文件的路径是否存在