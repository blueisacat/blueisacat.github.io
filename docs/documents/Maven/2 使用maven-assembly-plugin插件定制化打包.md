# 2 使用maven-assembly-plugin插件定制化打包

## 1 有个想法

以SpringBoot为例，使用官方默认的maven插件spring-boot-maven-plugin进行打包时，最终的形态是一个jar包，其中配置文件、lib依赖都会打进jar内，这就导致了整个jar包的体积很大。对于经常进行版本更迭的项目来说，每次都是要把整个jar上传。如果网速较慢的话，那么整个更迭的过程会很痛苦。

那么理想状态是什么样子呢？对于lib包、配置文件没有变更的情况，我只上传代码就可以了。也就是说如果将jar包中的配置文件、lib依赖分离出来，那么就可以更灵活的进行版本更迭了。

## 2 实现想法

有了想法，那么就开始实操啦。具体要实现的步骤如下：

1. 需要将jar进行瘦身，只把代码放进去，并添加对配置文件、lib依赖的引用；

2. 需要将需要的配置文件，拷贝出来；

3. 需要将lib依赖，拷贝出来；

4. 整合打包；

## 3 jar瘦身

jar瘦身，需要用到 `maven-jar-plugin` 插件，其配置如下：

* pom.xml

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>
            <archive>
                <manifest>
                    <!-- 项目启动类 -->
                    <mainClass>cn.blueisacat.gc.boot.Application</mainClass>
                    <!-- 依赖的jar的目录前缀 -->
                    <classpathPrefix>../lib</classpathPrefix>
                    <addClasspath>true</addClasspath>
                </manifest>
                <manifestEntries>
                    <!-- 配置文件的目录-->
                    <Class-Path>../config/</Class-Path>
                </manifestEntries>
            </archive>
            <excludes>
                <exclude>config/**</exclude>
            </excludes>
        </configuration>
    </plugin>
    ```

    !!! tip

        * mainClass：指定jar的启动类

        * classpathPrefix：添加lib依赖到classpath，`../lib` 为稍后lib依赖拷贝的目录

        * Class-Path：添加配置文件的目录到classpath，`../config` 为稍后配置文件拷贝的目录

## 4 配置文件拷贝

配置文件拷贝，需要用到 `maven-resources-plugin` 插件，其配置如下：

* pom.xml

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-resources-plugin</artifactId>
    </plugin>

    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <includes>
                <include>application.yml</include>
                <include>application-${profileActive}.yml</include>
                <include>logback-spring.xml</include>
            </includes>
            <targetPath>config</targetPath>
        </resource>
    </resources>
    ```

    !!! tip

        * directory：指定配置文件从哪里拷贝

        * targetPath：指定配置文件拷贝到哪里

        这里需要注意的是，最终拷贝到的根目录是 `\target\classes`  。

## 5 lib依赖拷贝

lib依赖拷贝，需要用到 `maven-dependency-plugin` 插件，其配置如下：

* pom.xml

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
            <execution>
                <phase>prepare-package</phase>
                <goals>
                    <goal>copy-dependencies</goal>
                </goals>
                <configuration>
                    <outputDirectory>target/lib</outputDirectory>
                    <overWriteReleases>false</overWriteReleases>
                    <overWriteSnapshots>false</overWriteSnapshots>
                    <overWriteIfNewer>true</overWriteIfNewer>
                    <includeScope>compile</includeScope>
                </configuration>
            </execution>
        </executions>
    </plugin>
    ```

    !!! tip

        * outputDirectory：lib依赖拷贝到哪里

## 6 整合打包

整合打包，需要用到 `maven-assembly-plugin` 插件，其配置如下：

* pom.xml

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <configuration>
            <descriptors>
                <descriptor>src/main/resources/assembly.xml</descriptor>
            </descriptors>
        </configuration>
        <executions>
            <execution>
                <id>make-assembly</id>
                <phase>package</phase>
                <goals>
                    <goal>single</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    ```

    !!! tip

        * descriptor：指定了 `assembly.xml` 的位置

* assembly.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <assembly>
        <!-- 可自定义，这里指定的是项目环境 -->
        <id>${project.version}-${profileActive}</id>
        <!-- 打包的类型，如果有N个，将会打N个类型的包 -->
        <formats>
            <format>tar.gz</format>
        </formats>
        <includeBaseDirectory>true</includeBaseDirectory>
        <fileSets>
            <!--
                0755->即用户具有读/写/执行权限，组用户和其它用户具有读写权限；
                0644->即用户具有读写权限，组用户和其它用户具有只读权限；
            -->
            <!-- 将src/bin目录下的所有文件输出到打包后的bin目录中 -->
            <fileSet>
                <directory>${basedir}/src/bin</directory>
                <outputDirectory>bin</outputDirectory>
                <fileMode>0755</fileMode>
                <includes>
                    <include>**.sh</include>
                    <include>**.bat</include>
                </includes>
            </fileSet>
            <!-- 指定输出target/classes/config中的配置文件到config目录中 -->
            <fileSet>
                <directory>${basedir}/target/classes/config</directory>
                <outputDirectory>config</outputDirectory>
                <fileMode>0644</fileMode>
            </fileSet>
            <!-- 将第三方依赖打包到lib目录中 -->
            <fileSet>
                <directory>${basedir}/target/lib</directory>
                <outputDirectory>lib</outputDirectory>
                <fileMode>0755</fileMode>
            </fileSet>
            <!-- 将项目启动jar打包到boot目录中 -->
            <fileSet>
                <directory>${basedir}/target</directory>
                <outputDirectory>boot</outputDirectory>
                <fileMode>0755</fileMode>
                <includes>
                    <include>${project.build.finalName}.jar</include>
                </includes>
            </fileSet>
            <!-- 创建license空目录 -->
            <fileSet>
                <outputDirectory>license</outputDirectory>
                <excludes>
                    <exclude>**/*</exclude>
                </excludes>
            </fileSet>
        </fileSets>
    </assembly>
    ```

## 7 验证想法

通过按照上面的插件配置，最终打包会在 `\target` 目录下生成一个 `*.tar.gz` 的压缩文件，解压后其目录结构如下：

* 根目录

    * boot

        * jar包

    * config

        * application.yml

        * application-dev.yml

        * logback-spring.xml

    * lib

        * 依赖.jar

        * ……

        * 依赖.jar

    * license

至此，最终形态已经符合预期。
