####  java.lang.IndexOutOfBoundsException

越界异常，出现在List集合获取元素时，`list.get(0)`,在使用前需要进行`list != null || list.size() != 0`的非空判断

#### Spring中的org.springframework.beans.factory.NoSuchBeanDefinitionException

![1580993181976](E:\Java\new_Java_Study\daily_notes\pictures\Spring创建工程的目录结构注意事项.png)

在bean.xml中，`<context:component-scan base-package="java"></context:component-scan>`，并<font color=ff0000>不能</font>读取整个项目下的注解

