---
title: Unpacking Maven Dependencies
date: 2012-08-07
---

### Unpacking a kit for inclusion in a Java library

The first time I came to do this I was new to maven and I didn't know 
about the maven-dependency-plugin, so I had a hard time keeping a handle
on the downloaded kit. Well, I'm back, and this time I mean business.

I've listed the plugins I used in something like sequential order:

### maven-dependency-plugin

The maven-dependency-plugin is used to download a kit which contains
a bunch of templates used in my artifact (which happens to be a Java 
library). I chose to bind this to the `process-resources` phase because 
nobody told me not to, but really it could have gone anywhere before the
antrun execution.

{% highlight xml %}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>2.4</version>
    <executions>
        <execution>
            <id>copy-dependency</id>
            <phase>process-resources</phase>
            <goals>
                <goal>unpack</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>

                        <groupId>com.mycompany</groupId>
                        <artifactId>some-artifact</artifactId>
                        <type>zip</type>
                        <classifier>some-classifier</classifier>
                        <version>some-version</version>
                        <overWrite>true</overWrite>
                        <outputDirectory>${project.build.directory}/template-kit</outputDirectory>
                        <excludes>**/*.inventory</excludes>

                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

### maven-antrun-plugin

I used antrun to unpack the kit to a staging directory. The working tree
in the staging directory is organised in the same way as in the jar. The
property `${project.build.directory}` is globally set and holds the 
location of the `target/` directory.

{% highlight xml %}
<plugin>
    <artifactId>maven-antrun-plugin</artifactId>
    <executions>
        <execution>
            <id>template-unpack</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <tasks>

                    <copy todir="${staging}">
                        <fileset dir="${project.build.directory}/template-kit/">
                            <include name="**/some/files/to/include/*" />
                        </fileset>
                        <flattenmapper />
                    </copy>

                </tasks>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

### maven-assembly-plugin

Finally the assembly plugin is is used in conjunction with the assembly
descriptor (`kit.xml`) to jar everything up.

{% highlight xml %}
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
            <configuration>
                <appendAssemblyId>false</appendAssemblyId>
                <archive>
                    <manifestFile>target/classes/META-INF/MANIFEST.MF</manifestFile>
                </archive>
                <descriptors>
                    <descriptor>src/main/assembly/kit.xml</descriptor>
                </descriptors>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

The `kit.xml` for all the magic looks like this:

{% highlight xml %}
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
  <id>distribution</id>
  <formats>
    <format>jar</format>
  </formats>
  <includeBaseDirectory>false</includeBaseDirectory>
  <fileSets>
    <fileSet>
        <directory>target/staging/</directory>
        <outputDirectory>/</outputDirectory>
        <includes>
            <include>templates/**/*</include>
        </includes>
        <excludes>
            <exclude>**/*.java</exclude>
        </excludes>
        <filtered>false</filtered>
    </fileSet>
  </fileSets>
</assembly>
{% endhighlight %}

Here's the documentation for the [maven-dependency-plugin](http://maven.apache.org/plugins/maven-dependency-plugin/).
It's got some jolly good [examples](http://maven.apache.org/plugins/maven-dependency-plugin/examples/copying-artifacts.html).
