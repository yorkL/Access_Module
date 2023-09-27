# 解决JDK17 模块访问的 unnamed module问题

# 报错原因
jdk9后增加了 module模块化概念，模块包增加了opens和export概念，需要暴露出去了才能调用。

# 解决方案
写个暴露模块项目,开放所有模块包.虽然不推荐开放，但是很多外部jar的引用都没有对应jdk版本，感觉想要所有都模块化的难度有点大。

## 核心代码如下：
```java
import jdk.internal.access.SharedSecrets;
 
/** @author qiuyi.l
 * @date 2022/7/13 14:24
 *<p>解决JDK module UNNAMED
 *<p>启动参数需要加上--add-exports java.base/jdk.internal.access=ALL-UNNAMED
 *<p>pom.xml需要配置编译参数--add-exports=java.base/jdk.internal.access=ALL-UNNAMED，不然maven编译会报错
 */
 
public class AccessModuleUtil {
    /**
     *开放所有模块(放到启动的main方法第一行调用。测过运行时调用不会生效)
     */
    public static void exportAll(){
        for (Module module : ModuleLayer.boot().modules()) {
            for (String pkgName : module.getPackages()){
                SharedSecrets.getJavaLangAccess().addOpensToAllUnnamed(module,pkgName);
                SharedSecrets.getJavaLangAccess().addExportsToAllUnnamed(module,pkgName);
            }
        }
    }
}
```
## maven添加编译参数：
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>org.example</groupId>
    <artifactId>accessmodule</artifactId>
    <version>1.0-SNAPSHOT</version>
 
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>utf-8</encoding>
                    <compilerArgs>
                        <arg>--add-exports=java.base/jdk.internal.access=ALL-UNNAMED</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
# git代码：
[GitHub - yorkL/Access_Module](https://github.com/yorkL/Access_Module.git)

idea编译报错,根据提示加入编译参数即可


# 使用建议：
独立一个小模块项目或者打包成jar进行引用，这样不必每次都添加编译参数
# 
————————————————
版权声明：本文为CSDN博主「york_csdn」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/york_csdn/article/details/133344250
