---
title: maven-shade-plugin使用中的小问题
date: 2022-04-22 23:17:08
categories: 杂记
tags: markdown模板
---

# maven-shade-plugin使用中的小问题

maven-shade-plugin打包导致META-INF/SERVICES中的文件丢失,导致jar在动态加载时无法启动.这个问题困扰了我一周,暂时先记录处理办法,后续完善解决步骤

主要是依据这篇文章来解决处理的
https://cloud.tencent.com/developer/article/1622207


- maven-shade-plugin模板

```xml

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.1</version>
            <configuration>
                <!-- put your configurations here -->
                <!--只包含该项目代码中用到的jar,在父项目中引入了，但在当前模块中没有用到就会被删掉-->
                <minimizeJar>true</minimizeJar>
                <!--重新定位类位置，就好像类是自己写的一样，修改别人jar包的package-->
                <relocations>
                    <relocation>
                        <pattern>com.alibaba.fastjson</pattern>
                        <shadedPattern>com.gavinzh.learn.fastjson</shadedPattern>
                        <excludes>
                            <!--这些类和包不会被改变-->
                            <exclude>com.alibaba.fastjson.not.Exists</exclude>
                            <exclude>com.alibaba.fastjson.not.exists.*</exclude>
                        </excludes>
                    </relocation>
                </relocations>
            </configuration>
            <executions>
                <execution>
                    <configuration>
                        <!--创建一个你自己的标识符，位置在原有名称之后-->
                        <shadedArtifactAttached>true</shadedArtifactAttached>
                        <shadedClassifierName>gavinzh</shadedClassifierName>
                        <!--在打包过程中对文件做一些处理工作-->
                        <transformers>
                            <!--在META-INF/MANIFEST.MF文件中添加key: value 可以设置Main方法-->
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <manifestEntries>
                                    <mainClass>com.gavinzh.learn.shade.Main</mainClass>
                                    <Build-Number>123</Build-Number>
                                    <Built-By>your name</Built-By>
                                    <X-Compile-Source-JDK>1.7</X-Compile-Source-JDK>
                                    <X-Compile-Target-JDK>1.7</X-Compile-Target-JDK>
                                </manifestEntries>
                            </transformer>
                            <!--阻止META-INF/LICENSE和META-INF/LICENSE.txt-->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer"/>
                            <!--合并所有notice文件-->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheNoticeResourceTransformer">
                                <addHeader>true</addHeader>
                            </transformer>
                            <!--如果多个jar包在META-INF文件夹下含有相同的文件，那么需要将他们合并到一个文件里-->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.handlers</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.schemas</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.factories</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.tld</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring-form.tld</resource>
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.tooling</resource>
                            </transformer>
                            <!--如果多个jar包在META-INF文件夹下含有相同的xml文件，则需要聚合他们-->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ComponentsXmlResourceTransformer"/>
                            <!--排除掉指定资源文件-->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.DontIncludeResourceTransformer">
                                <resource>.no_need</resource>
                            </transformer>
                            <!--将项目下的文件file额外加到resource中-->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.IncludeResourceTransformer">
                                <resource>META-INF/pom_test</resource>
                                <file>pom.xml</file>
                            </transformer>
                            <!--整合spi服务中META-INF/services/文件夹的相关配置-->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                        </transformers>
                    </configuration>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```