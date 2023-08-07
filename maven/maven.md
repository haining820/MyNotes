---
title: maven整理
date: 2022-08-14
tags: [maven]
toc: true
---

> https://blog.csdn.net/weixin_38569499/article/details/91456988

# 0、命令速查

修改 settings.xml 之后，无法加载包，使用该命令强制 Maven 重新加载配置文件并重新安装依赖项

```
mvn clean install -U
```



# 1、project

project 是 pom 文件的根元素，project 下有 modelVersion、groupId、artifactId、version、packaging 等重要元素，使用 groupId、artifactId、version 三个元素定义一个项目。

<!--more-->

<font size=4 style="font-weight:bold;background:yellow;">头信息（粘贴即用）</font>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
</project>
```

`project`：根元素；

`xmlns`：命名空间，类似包名
`xmlns:xsi`：xml 遵循的标签规范
`xsi:schemaLocation`：用来定义 xmlschema 的地址，也就是xml书写时需要遵循的语法

<font size=4 style="font-weight:bold;background:yellow;">字段说明</font>

```xml
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.haining820</groupId>
    <artifactId>study-projects</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
```

`modelVersion`：声明项目描述符遵循哪一个 pom 模型版本，模型本身的版本很少改变，所以 modelVersion 一般固定不变；

`groupId`：公司或者组织的唯一标志，并且配置时生成的路径也是由此生成，如 com.trip.finance，maven 会将该项目打成的 jar 包放本地路径：/com/trip/finance；

`artifactId`：本项目的唯一 id，一个 groupId 下面可能多个项目，就是靠 artifactId 来区分的；

`version`：本项目目前所处的版本号；

`name`：项目的名称，Maven 产生的文档用（可省略）；

`packaging`  打包类型，可取值：pom、jar、maven-plugin、ejb、war、ear、rar、par 等等；

> https://blog.csdn.net/imaginehero/article/details/103706732

- `<packaging>pom</packaging>`

  在父级项目中的 pom.xml 文件使用的 packaging 配置一定为 pom。父级的pom文件只作项目的子模块的整合，在 maven install 时不会生成 jar/war 压缩包。

- `<packaging>jar</packaging>`

  jar 包是最为常见的打包方式，当 pom 文件中没有设置 packaging 参数时，默认使用 jar 方式打包；
  这种打包方式意味着在 maven build 时会将这个项目中的所有 java 文件都进行编译形成 `.class` 文件，且按照原来的 java 文件层级结构放置，最终压缩为一个 jar 文件。
  当使用 `mvn install` 命令的时候，能够发现在项目中与 src 文件夹同级新生成了一个 target 文件夹，这个文件夹内的 classes 文件夹即为刚才提到的编译后形成的文件夹。如下图所示，这是我自己的项目生成的 target 文件夹，而最下方的 jar 文件即为此文件夹的压缩版本。

- `<packaging>war</packaging>`

  war 包与 jar 包非常相似，同样是编译后的 .class 文件按层级结构形成文件树后打包形成的压缩包。不同的是，它会将项目中依赖的所有 jar 包都放在 `WEB-INF/lib` 这个文件夹下。

# 2、parent

父项目的坐标，坐标包括 groupId，artifactId 和 version，如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

`groupId` ：父项目的 groupId，用于定位父项目；

`artifactId` ：父项目的 artifactId，用于定位父项目；

`version` ：父项目的 version，用于定位父项目；

`relativePath`：

- 指定查找该父项目 pom.xml 的(相对)路径。默认顺序：构建当前项目的地方 > relativePath > 本地仓库 > 远程仓库；
- 没有 relativePath 标签等同 `../pom.xml`，即默认从当前 pom 文件的上一级目录找，表示不从 relativePath 找，直接从本地仓库找，找不到再从远程仓库找。

# 3、properties

为 pom 定义一些常量，在 pom 中的其它地方可以直接引用。

```xml
 <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>

        <spring.version>4.3.0.RELEASE</spring.version>
        <mybatis-spring.version>1.3.0</mybatis-spring.version>

        <slf4j.version>1.7.5</slf4j.version>
        <junit.version>4.12</junit.version>
        <guava.version>20.0</guava.version>

        <mysql-connector.version>5.1.36</mysql-connector.version>
        <mybatis.version>3.3.0</mybatis.version>

        <dubbo.version>2.7.3</dubbo.version>
        <servlet-api.version>2.5</servlet-api.version>
        <joda-time.version>2.10.10</joda-time.version>

    </properties>
```

# 4、dependencyManagement

dependencyManagement 元素提供了一种管理依赖版本号的方式。在 dependencyManagement 元素中声明所依赖的 jar 包的版本号等信息，那么所有子项目再次引入此依赖 jar 包时则无需显式的列出版本号。**Maven 会沿着父子层级向上寻找拥有 dependencyManagement 元素的项目，然后使用它指定的版本号。**

- pom 中在该标签下声明依赖**只起到一个声明的作用，实际上并不会真实导入；**

- 当子模块 pom 需要引入对应依赖时，只需声明对应的 groupId 和 artifactId 即可，省略掉的 version 和 scope，会沿着树向上层 pom 查找最近的 dependencyManagement 声明的 groupId 和 artifactId，并使用其声明的 version 和 scope；

  如果有多个子项目都引用同一样依赖，可以避免在每个使用的子项目里都声明一个版本号。当想升级或切换到另一个版本时，只需要在顶层父容器里更新，不需要逐个修改子项目；

- 如果某个子项目需要另外的一个版本，也可以声明 version 和 scope， **覆盖父级继承的配置 dependencies 优先级高于 dependencyManagement；**

- 父 pom 中的 dependencyManagement 允许被子模块的 dependencyManagement 覆盖。

```xml
    <dependencyManagement>
        <dependencies>
            <!--参考dependencies/dependency元素 -->
            <dependency>
                ......
            </dependency>
        </dependencies>
    </dependencyManagement>
```

# 5、dependencies 标签

- pom 文件中通过 dependencyManagement 来声明依赖，通过 dependencies 元素来管理依赖。

- dependencies 子元素的配置和 dependencyManagement 下的 dependencices 相同；

- 而 dependencies 下只有一类直接的子元素：dependency，一个 dependency 子元素表示一个依赖项目。

<font size=4 style="font-weight:bold;background:yellow;">依赖坐标</font>

groupId/artifactId/version：依赖项目的坐标三元素，一个 maven 仓库中，完全相同的一组 groupId、artifactId、version，只能有一个项目。

**依赖范围**

provided：在运行期外部已经提供了，所以在运行时就不需要

可能会与依赖包有一些交互，spring 接口controller拿到请求获取对象，httpservletresponse、httpservletrequest，但是要导入servlet，写的时候要引进来但是在编译期不需要，运行时候不需要这个

jdbc驱动：写的时候不需要，但是运行时需要

| 依赖范围 | 编译有效 | 测试有效 | 运行有效 |           例子            |
| :------: | :------: | :------: | :------: | :-----------------------: |
| compile  |    ✔     |    ✔     |    ✔     |        Spring-core        |
|   test   |    -     |    ✔     |    -     |           Junit           |
| provided |    ✔     |    ✔     |    -     |        Servlet-api        |
| runtime  |    -     |    ✔     |    ✔     |           JDBC            |
|  system  |    ✔     |    ✔     |    -     | 本地 maven 仓库以外的 jar |

<font size=4 style="font-weight:bold;background:yellow;">依赖排除</font>

```xml
    <!--该元素描述了项目相关的所有依赖。 这些依赖自动从项目定义的仓库中下载 -->
    <dependencies>
        <dependency>
            <!------------------- 依赖坐标 ------------------->
            <!--依赖项目的坐标三元素：groupId + artifactId + version -->
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-artifact</artifactId>
            <version>3.8.1</version>
 
            <!------------------- 依赖类型 ------------------->
            <!-- 依赖类型，默认是jar。通常表示依赖文件的扩展名，但有例外。一个类型可以被映射成另外一个扩展名或分类器 -->
            <!-- 类型经常和使用的打包方式对应，尽管这也有例外，一些类型的例子：jar，war，ejb-client和test-jar -->
            <!-- 如果设置extensions为true，就可以在plugin里定义新的类型 -->
            <type>jar</type>
            <!-- 依赖的分类器。分类器可以区分属于同一个POM，但不同构建方式的构件。分类器名被附加到文件名的版本号后面 -->
            <!-- 如果想将项目构建成两个单独的JAR，分别使用Java 4和6编译器，就可以使用分类器来生成两个单独的JAR构件 -->
            <classifier></classifier>
 
            <!------------------- 依赖传递 ------------------->
            <!--依赖排除，即告诉maven只依赖指定的项目，不依赖该项目的这些依赖。此元素主要用于解决版本冲突问题 -->
            <exclusions>
                <exclusion>
                    <artifactId>spring-core</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
            <!-- 可选依赖，用于阻断依赖的传递性。如果在项目B中把C依赖声明为可选，那么依赖B的项目中无法使用C依赖 -->
            <optional>true</optional>
            
            <!------------------- 依赖范围 ------------------->
            <!--依赖范围。在项目发布过程中，帮助决定哪些构件被包括进来
                - compile：默认范围，用于编译;  - provided：类似于编译，但支持jdk或者容器提供，类似于classpath 
                - runtime: 在执行时需要使用;    - systemPath: 仅用于范围为system。提供相应的路径 
                - test: 用于test任务时使用;    - system: 需要外在提供相应的元素。通过systemPath来取得 
                - optional: 当项目自身被依赖时，标注依赖是否传递。用于连续依赖时使用 -->
            <scope>test</scope>
            
            <!-- 该元素为依赖规定了文件系统上的路径。仅供scope设置system时使用。但是不推荐使用这个元素 -->
            <!-- 不推荐使用绝对路径，如果必须要用，推荐使用属性匹配绝对路径，例如${java.home} -->
            <systemPath></systemPath>
            
        </dependency>
    </dependencies>
```

<font size=4 style="font-weight:bold;background:yellow;">版本号</font>

```java
3.5.1-SNAPSHOT	// 不稳定版本, 用于开发调试
3.5.1-PRE		// 灰度版本
3.5.1			// RELEASE或不带后缀：正式版本
```

```
<主版本>-<次版本>-<增量版本> -> <里程碑版本>
```

**主版本** 表示了项目的重大架构变更

**次版本** 表示较大范围的功能添加和变化

**增量版本** 日常bug修复或小需求发布

**里程碑版本号** 版本指某一个版本号的里程碑

# 6、依赖冲突

<font size=4 style="font-weight:bold;background:yellow;">依赖传递</font>

```
C -> A(1.0.0),  
C->B ->A(2.0.0)
```

依赖 A 出现冲突，1.0.0 or 2.0.0 ？

- **短路径者优先**

  ```java
  A—>B—>C—>D—>E—>X(version 0.0.1)
  A—>F—>X(version 0.0.2)			// A依赖于X(version 0.0.2)
  ```

- **先声明者优先（顺序）**

  ```java
  A—>E—>X(version 0.0.1)
  A—>F—>X(version 0.0.2)			// 哪个在前面依赖哪个
  ```

<font size=4 style="font-weight:bold;background:yellow;">依赖排除</font>

公司规范：要显式的定义出版本号，尽量使用 `<dependencyManagement>`，`<exclusions>`仅用于处理不同 jar 相同类名的冲突

# 7、build

build 标签定义了构建项目需要的信息

路径管理、资源管理、插件管理、构建扩展、其他配置

```xml
    <!--构建项目需要的信息 -->
    <build>
        <!--------------------- 路径管理（在遵循约定大于配置原则下，不需要配置） --------------------->
        <!--项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。 -->
        <sourceDirectory />
        <!--该元素设置了项目单元测试使用的源码目录。该路径是相对于pom.xml的相对路径 -->
        <testSourceDirectory />
        <!--被编译过的应用程序class文件存放的目录。 -->
        <outputDirectory />
        <!--被编译过的测试class文件存放的目录。 -->
        <testOutputDirectory />        
        <!--项目脚本源码目录，该目录下的内容，会直接被拷贝到输出目录，因为脚本是被解释的，而不是被编译的 -->
        <scriptSourceDirectory />
 
        <!--------------------- 资源管理 --------------------->
        <!--这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里。 -->
        <resources>
            <!--这个元素描述了项目相关或测试相关的所有资源路径 -->
            <resource>
                <!-- 描述了资源的目标输出路径。该路径是相对于target/classes的路径 -->
                <!-- 如果是想要把资源直接放在target/classes下，不需要配置该元素 -->
                <targetPath />
                <!--是否使用参数值代替参数名。参数值取自文件里配置的属性，文件在filters元素里列出。 -->
                <filtering />
                <!--描述存放资源的目录，该路径相对POM路径 -->
                <directory />
                <!--包含的模式列表，例如**/*.xml，只有符合条件的资源文件才会在打包的时候被放入到输出路径中 -->
                <includes />
                <!--排除的模式列表，例如**/*.xml，符合的资源文件不会在打包的时候会被过滤掉 -->
                <excludes />
            </resource>
        </resources>
        <!--这个元素描述了单元测试相关的所有资源路径，例如和单元测试相关的属性文件。 -->
        <testResources>
            <!--这个元素描述了测试相关的所有资源路径，子元素说明参考build/resources/resource元素的说明 -->
            <testResource>
                <targetPath />
                <filtering />
                <directory />
                <includes />
                <excludes />
            </testResource>
        </testResources>
 
        <!--------------------- 插件管理 --------------------->
        <!-- 子项目可以引用的默认插件信息。pluginManagement中的插件直到被引用时才会被解析或绑定到生命周期 -->
        <!-- 这里只是做了声明，并没有真正的引入。给定插件的任何本地配置都会覆盖这里的配置-->
        <pluginManagement>
            <!-- 可使用的插件列表 -->
            <plugins>
                <!--plugin元素包含描述插件所需要的信息。 -->
                <plugin>
                    <!--插件在仓库里的group ID -->
                    <groupId />
                    <!--插件在仓库里的artifact ID -->
                    <artifactId />
                    <!--被使用的插件的版本（或版本范围） -->
                    <version />
                    <!-- 是否从该插件下载Maven扩展(例如打包和类型处理器) -->
                    <!-- 由于性能原因，只有在真需要下载时，该元素才被设置成enabled -->
                    <extensions />
                    <!--在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。 -->
                    <executions>
                        <!--execution元素包含了插件执行需要的信息 -->
                        <execution>
                            <!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标 -->
                            <id />
                            <!--绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段 -->
                            <phase />
                            <!--配置的执行目标 -->
                            <goals />
                            <!--配置是否被传播到子POM -->
                            <inherited />
                            <!--作为DOM对象的配置 -->
                            <configuration />
                        </execution>
                    </executions>
                    <!--项目引入插件所需要的额外依赖 -->
                    <dependencies>
                        <!--参见dependencies/dependency元素 -->
                        <dependency>
                            ......
                        </dependency>
                    </dependencies>
                    <!--任何配置是否被传播到子项目 -->
                    <inherited />
                    <!--作为DOM对象的配置 -->
                    <configuration />
                </plugin>
            </plugins>
        </pluginManagement>
        <!--使用的插件列表 -->
        <plugins>
            <!--参见build/pluginManagement/plugins/plugin元素 -->
            <plugin>
                <groupId />
                <artifactId />
                <version />
                <extensions />
                <executions>
                    <execution>
                        <id />
                        <phase />
                        <goals />
                        <inherited />
                        <configuration />
                    </execution>
                </executions>
                <dependencies>
                    <!--参见dependencies/dependency元素 -->
                    <dependency>
                        ......
                    </dependency>
                </dependencies>
                <goals />
                <inherited />
                <configuration />
            </plugin>
        </plugins>
 
        <!--------------------- 构建扩展 --------------------->
        <!--使用来自其他项目的一系列构建扩展 -->
        <extensions>
            <!--每个extension描述一个会使用到其构建扩展的一个项目，extension的子元素是项目的坐标 -->
            <extension>
                <!--项目坐标三元素：groupId + artifactId + version -->
                <groupId />
                <artifactId />
                <version />
            </extension>
        </extensions>
 
        <!--------------------- 其他配置 --------------------->
        <!--当项目没有规定目标（Maven2 叫做阶段）时的默认值 -->
        <defaultGoal />
        <!--构建产生的所有文件存放的目录 -->
        <directory />
        <!--产生的构件的文件名，默认值是${artifactId}-${version}。 -->
        <finalName />
        <!--当filtering开关打开时，使用到的过滤器属性文件列表 -->
        <filters />
    </build>
```
