# mill-assembly-issue
Mill 0.11.1 assembly issue demo

## Env

### Java
```
> java --version                                                                                                                         
openjdk 11.0.19 2023-04-18
OpenJDK Runtime Environment Temurin-11.0.19+7 (build 11.0.19+7)
OpenJDK 64-Bit Server VM Temurin-11.0.19+7 (build 11.0.19+7, mixed mode)

```

### Scala
```
scala --version                                                                                                                          Scala code runner version 2.13.11 -- Copyright 2002-2023, LAMP/EPFL and Lightbend, Inc.

```

### Mill
```
./mill version                                                                                                                           [1/1] version 
0.11.0-19-d4e35f
```

### Description

Try to create an assembly of a project to run it with `java -jar foo.jar` but fail
I reproduce a bug I get in my project with a simple project and fiew but not small dependency.

The issue is when you add at the same time:
- `ivy"org.apache.spark::spark-core:3.4.0"`
- `ivy"dev.zio::zio:2.0.15"` or `ivy"org.typelevel::cats-core:2.9.0"`

When you run the following commands:
```
> ./mill foo.assembly
[build.sc] [41/49] compile 
[info] compiling 1 Scala source to xxx ...
[info] done compiling
[47/47] foo.assembly 
> java -jar out/foo/assembly.dest/out.jar --text tutu
Error: An unexpected error occurred while trying to open file out/foo/assembly.dest/out.jar
```

If you try the following it will work:
```
./mill foo.run --text tutu
[48/48] foo.run 
<h1>tutu</h1>
```

If you try the following to understand you get this:
```
java -cp out/foo/assembly.dest/out.jar ultra.Main
Error: Could not find or load main class ultra.Main
Caused by: java.lang.ClassNotFoundException: ultra.Main
```

While the class is in the jar
```
> unzip -l out/foo/assembly.dest/out.jar | grep ultra
error: End-of-centdir-64 signature not where expected (prepended bytes?)
  (attempting to process anyway)
warning [out/foo/assembly.dest/out.jar]:  209 extra bytes at beginning or within zipfile
  (attempting to process anyway)
        0  2023-07-04 09:28   ultra/
     6115  2023-07-04 09:28   ultra/Main$.class
     1075  2023-07-04 09:28   ultra/Main.class
```

Maybe you will notice the strange message from unzip maybe it's specific of the jars?
What I can say is that the working version has just the `warning`
```
error: End-of-centdir-64 signature not where expected (prepended bytes?)
  (attempting to process anyway)
warning [out/foo/assembly.dest/out.jar]:  209 extra bytes at beginning or within zipfile
```

Then finally if you try to list the content of the jar with the `jar` command it fail
```
> jar --list -f out/foo/assembly.dest/out.jar
java.util.zip.ZipException: invalid CEN header (bad signature)
        at java.base/java.util.zip.ZipFile$Source.zerror(ZipFile.java:1607)
        at java.base/java.util.zip.ZipFile$Source.initCEN(ZipFile.java:1557)
        at java.base/java.util.zip.ZipFile$Source.<init>(ZipFile.java:1308)
        at java.base/java.util.zip.ZipFile$Source.get(ZipFile.java:1271)
        at java.base/java.util.zip.ZipFile$CleanableResource.<init>(ZipFile.java:733)
        at java.base/java.util.zip.ZipFile$CleanableResource.get(ZipFile.java:850)
        at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:248)
        at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:177)
        at java.base/java.util.zip.ZipFile.<init>(ZipFile.java:148)
        at jdk.jartool/sun.tools.jar.Main.list(Main.java:1499)
        at jdk.jartool/sun.tools.jar.Main.run(Main.java:380)
        at jdk.jartool/sun.tools.jar.Main.main(Main.java:1680)
```

BTW This works with `sbt` assembly
