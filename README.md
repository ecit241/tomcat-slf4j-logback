# Tomcat + SLF4J + Logback #

(upstream is https://github.com/grgrzybek/tomcat-slf4j-logback)

Tomcat, by default, includes a version of [Apache Commons Logging](http://commons.apache.org/proper/commons-logging/), withi the package names changed from `org.apache.commons.logging` to `org.apache.juli.logging`, and hardcoded to use java.util.logging (http://tomcat.apache.org/tomcat-7.0-doc/logging.html#Using_Log4j), at /usr/share/tomcat7/bin/tomcat-juli.jar. 

SLF4J has an [adapter library](http://www.slf4j.org/legacy.html#jclOverSLF4J) that you can use to send logging calls against against the Apache Commons Logging API over to SLF4j.  You just replace commons-logging.jar with jcl-over-slf4j.jar, and you still have classes under `org.apache.commons.logging` that let you call `log.info` or whatever, only they're evil imposter classes that happen to share the same name and API as the Apache classes, but they're from SLF4J.  

I hope you can see where this is going by now.  What this repo does, is take SLF4J's adapter library, rename `org.apache.commons.logging` to `org.apache.juli.logging`, and put it at /usr/share/tomcat7/bin/tomcat-juli.jar. Viola! Tomcat is logging over SLF4J, none the wiser.

## Implementation details (follow along in build.xml):

Line number. Explanation

24. Unzip the existing tomcat-juli.jar (we still need some classes from it - ClassLoaderLogManager*.class and UserDataHelper*.class (tomcat-specific additions to commons-logging.jar)
30. Search and replace org.apache.commons to org.apache.juli
37. Compile the whole mess
53. Jar the mess.

This jarring step has an interesting addition.  By default, when tomcat's booting, only tomcat-juli.jar (among its other tomcat stuff) is on the classpath.  We've swapped in a renamed version of jcl-over-slf4j.jar for that, but we still want to depend on slf4j and logback, now.  How to get those on the classpath?  In the META-INF/MANIFEST.MF of the jar, you can specify additions to the classpath:
```
Class-Path: /var/lib/tomcat7/common/slf4j-api-1.7.5.jar /var/lib/tomca
 t7/common/logback-core-1.0.12.jar /var/lib/tomcat7/common/logback-cla
 ssic-1.0.12.jar /var/lib/tomcat7/common/classes/
```
They're in /var/lib/tomcat7/common, so they'll already be on the classpath for our deployed war.  Also, /var/lib/tomcat7/common/classes/ is added, so our logback.xml in there will be on the classpath when logback starts up.

## How to build
* Get a tomcat-juli.jar from somewhere.  Either copy it out of the tomcat install, or download one from [maven central](http://repo1.maven.org/maven2/org/apache/tomcat/tomcat-juli/7.0.21/tomcat-juli-7.0.21.jar)
* Put it in `/_external` in this directory.
* Run `ant`

The output should be in `_dist/tomcat-juli.jar`
