# :fontawesome-solid-file-word: Dockerfile最佳实践

## 减少构建时间
> Key：利用缓存特性来减少构建时间

### Dockerfile指令顺序
构建步骤（Dockerfile指令）的顺序很重要，因为当通过更改文件或修改Dockerfile中的行使步骤的缓存无效时，后续缓存的步骤将中断。 **将您的步骤从最少到最频繁更改的步骤排序，以优化缓存。**

=== "Bad"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    COPY . /home
    RUN apt-get update
    RUN apt-get install -y openjdk-8-jdk ssh
    CMD ["java","-jar","/home/target/app.jar"]
    ```

=== "Good"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update
    RUN apt-get install -y openjdk-8-jdk ssh
    COPY . /home
    CMD ["java","-jar","/home/target/app.jar"]
    ```

### 更细的复制行为
**仅复制所需内容**。 如果可能，请避免“复制”。 将文件复制到图像中时，请确保对要复制的内容非常明确。 对要复制的文件的任何更改都将破坏缓存。

=== "Bad"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update
    RUN apt-get install -y openjdk-8-jdk ssh
    COPY . /home
    CMD ["java","-jar","/home/target/app.jar"]
    ```

=== "Good"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update
    RUN apt-get install -y openjdk-8-jdk ssh
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

### 识别可缓存单元
每个RUN指令都可以看作是可缓存的执行单元。 它们过多可能是不必要的，而将所有命令链接到一条RUN指令可能会轻易破坏高速缓存，从而损害开发周期。 从程序包管理器安装程序包时，您始终希望更新索引并以相同的RUN模式安装程序包：它们一起形成一个可缓存单元。 否则，您将冒着安装过时的软件包的风险。

=== "Bad"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update
    RUN apt-get install -y openjdk-8-jdk ssh
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

=== "Good"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update && apt-get install -y openjdk-8-jdk ssh
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

## 减少镜像体积
映像大小可能很重要，因为较小的映像等于更快的部署和较小的攻击。

### 删除不必要的依赖
删除不必要的依赖项，并且不要安装调试工具。 如果需要，以后可以始终安装调试工具。 某些软件包管理器（如apt）会自动安装用户指定的软件包推荐的软件包，从而不必要地增加了占用空间。 Apt具有--no-install-recommends标志，可确保不安装实际上不需要的依赖项。 如果需要它们，请显式添加它们。

=== "Bad"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update && \
        apt-get install -y openjdk-8-jdk ssh vim
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

=== "Good"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update && \
        apt-get install -y openjdk-8-jdk
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

### 删除程序包管理器缓存
程序包管理器维护自己的缓存，该缓存可能最终出现在映像中。 一种解决方法是在安装软件包的同一条RUN指令中删除缓存。 在另一个RUN指令中将其删除不会减小image大小。

=== "Bad"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update && \
        apt-get install -y openjdk-8-jdk
    RUN rm -rf /xxxx/xxxxxx/xxxx
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

=== "Good"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update && \
        apt-get install -y openjdk-8-jdk && \
        rm -rf /xxxx/xxxxxx/xxxx
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

## 可维护性

### 尽可能使用官方镜像
官方映像可以节省大量维护时间，因为所有安装步骤都已完成，并且采用了最佳实践。 如果您有多个项目，则他们可以共享这些图层，因为它们使用的是完全相同的基本图像。

=== "Bad"
    ``` Dockerfile
    FROM debian
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    RUN apt-get update && \
        apt-get install -y openjdk-8-jdk && \
        rm -rf /xxxx/xxxxxx/xxxx
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

=== "Good"
    ``` Dockerfile
    FROM openjdk:latest
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```


### 使用指定的tag
不要使用latest标签。 latest对于始终可用于Docker Hub上的官方映像比较便利，实际使用过程中，如果有重大更改迭代，可能会因为版本问题，导致Dockerfile构建失败。

=== "Bad"
    ``` Dockerfile
    FROM openjdk:latest
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

=== "Good"
    ``` Dockerfile
    FROM openjdk:8
    MAINTAINER weeknd.su@gmail.com
    LABEL app='jdk'
    COPY target/app.jar /home
    CMD ["java","-jar","/home/app.jar"]
    ```

### 使用满足需求的最小镜像（minimal flavors）


## 重现性
保证Dockfile是在一个环境内构建的。例如。如果不同镜像版本的jar包是来自不同环境构建的。则失去了容器一致性环境的优点。

### 在一致的环境中从源代码构建
源代码是您要构建Docker映像的真实来源。 Dockerfile只是蓝图。
首先确定构建应用程序所需的所有内容。 我们简单的Java应用程序需要Maven和JDK，因此让我们的Dockerfile基于来自Docker Hub（包括JDK）的特定最小官方Maven映像。 如果需要安装更多的依赖项，则可以在RUN步骤中进行。
我们解决了不一致的环境问题，但又引入了另一个问题：每次更改代码时，都会提取pom.xml中描述的所有依赖项。

``` Dockerfile
FROM ant:3.6.3-openjdk-16
COPY app /home
RUN ant -buildfile /home/text.xml
```

```
  docker cp <container id>:/home/target/app.jar .
```

``` Dockerfile
FROM openjdk:8
MAINTAINER weeknd.su@gmail.com
LABEL app='jdk'
COPY app/app.jar /home
CMD ["java","-jar","/home/app.jar"]
```


### 在单独的步骤中获取依赖项
通过再次考虑可缓存的执行单元，我们可以确定获取依赖项是一个单独的可缓存单元，只需要依赖对pom.xml的更改，而不必依赖于源代码。 两个COPY步骤之间的RUN步骤告诉Maven仅获取依赖项。

在一致的环境中进行构建会引入另一个问题：我们的映像比以前大得多，因为它包含了运行时不需要的所有构建时依赖性。

### 使用多阶段（multi-stages）构建来删除构建依赖关系
多个FROM语句可识别多阶段构建。 每个FROM都开始一个新阶段。 可以使用AS关键字来命名它们，我们使用该关键字来命名第一阶段的“构建器”，以供以后参考。 它将包括我们在一致环境中的所有构建依赖项。

第二阶段是我们的最后阶段，它将产生最终的图像。 它将包括运行时所必需的严格条件，在这种情况下，它是基于Alpine的最小JRE（Java运行时）。 中间生成器阶段将被缓存，但不会出现在最终映像中。 为了使构建工件进入最终图像，请使用COPY --from = STAGE_NAME。 在这种情况下，STAGE_NAME是构建者。

多阶段构建是消除构建时依赖关系的首选解决方案。

我们从不一致地构建肿的映像到在一致的环境中构建对缓存友好的最小映像。


``` Dockerfile
FROM ant:3.6.3-openjdk-16 as buildwar
COPY app /home
RUN ant -buildfile /home/text.xml

FROM openjdk:8
MAINTAINER weeknd.su@gmail.com
LABEL app='jdk'
COPY --from=buildwar /home/target/app.jar /home
CMD ["java","-jar","/home/app.jar"]
```