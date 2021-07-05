资料参考：https://blog.csdn.net/IT_Migrant_worker/article/details/104743218

其它问题：

```java
import com.sun.tools.javac.api.BasicJavacTask;
import com.sun.tools.javac.processing.JavacProcessingEnvironment;
import com.sun.tools.javac.util.Context;
```

JavacTask类找不到该类。

解决方式：将jdk/lib/tools.jar导入到项目的Libraries，mac系统同理；

