---
title: Deploying an XML File with Maven
date: 2012-11-06
---

Something clever we've started doing at work is publishing common build
files to our local maven repository, so builds can depend on known versions
of scripts, rather than just the latest from source control.

Setting this up for the simplest case, involves uploading a single file,
for instance an ant script, to the repository. The best way I've seen
this done uses the build-helper plugin, which &mdash; amongst many other things &mdash;
can be used to deploy any file as the primary build artifact.

To deploy a script using the plugin I added the following to the `<plugins>`
element of my pom.

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>build-helper-maven-plugin</artifactId>
  <version>1.7</version>
  <executions>
    <execution>
      <phase>package</phase>
      <id>attach-artifacts</id>
      <goals>
        <goal>attach-artifact</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <artifacts>
      <artifact>
        <file>myBuildScript</file>
        <type>xml</type>
      </artifact>
    </artifacts>
  </configuration>
</plugin>
```

And run with `mvn deploy` to publish the artifact.

The [build-helper-maven-plugin](http://mojo.codehaus.org/build-helper-maven-plugin/index.html)
is documented at the Mojo Project.
