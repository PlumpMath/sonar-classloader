# Toolbox for Java Classloaders

Sonar Classloader is a toolbox for creating Java 7+ classloaders. It's inspired from projects [Codehaus Classworlds][classworlds] and
[Plexus Classworlds][plexus].

This library is available under GNU LGPLv3.

## Maven Dependency

    <dependency>
      <groupId>org.codehaus.sonar-plugins</groupId>
      <artifactId>sonar-classloader</artifactId>
      <version>${classloader.version}</version>
    </dependency>

## Usage

#### Build classloader

Create a classloader based on system classloader.

```java
ClassloaderBuilder builder = new ClassloaderBuilder();
Map<String, ClassLoader> classloaders = builder
  .newClassloader("the-cl")
  .addURL("the-cl", jarFile)
  .addURL("the-cl", directory)
  .build();

// this classloader can load only JRE and the resources contained in jarFile and directory. 
ClassLoader c = classloaders.get("the-cl");
```

It's also possible to create a classloader based on another one:

```java
ClassloaderBuilder builder = new ClassloaderBuilder();
Map<String, ClassLoader> classloaders = builder
  .newClassloader("the-cl", otherClassloader)
  .addURL("the-cl", jarFile)
  .build();

// this classloader can load the resources of JRE, jarFile and otherClassloader. 
ClassLoader cl1 = classloaders.get("cl1");
```

#### Hierarchy of classloaders

```java
ClassloaderBuilder builder = new ClassloaderBuilder();
Map<String, ClassLoader> classloaders = builder
  .newClassloader("the-parent")
  .addURL("the-parent", parentJar)
  
  .newClassloader("the-child")
  .addURL("the-child", childJar)
  .setParent("the-child", "the-parent", new Mask())
  
  .newClassloader("the-grand-child")
  .setParent("the-grand-child", "the-child", new Mask())
  // can be parent-first or self-first ordering strategy. Default is parent-first.
  .setLoadingOrder("the-grand-child", LoadingOrder.SELF_FIRST)
  
  .build();
ClassLoader parent = classloaders.get("the-parent");
ClassLoader child = classloaders.get("the-child");
ClassLoader grandChild = classloaders.get("the-grand-child");
```

Importing classes and resources from sibling classloaders allows to convert the standard tree of classloaders to a graph a classloaders.

```java
ClassloaderBuilder builder = new ClassloaderBuilder();
// build 4 classloaders. "the-child" searches for resources from sibling1, sibling2, the-parent then itself.
Map<String, ClassLoader> classloaders = builder
  .newClassloader("the-parent")
  .newClassloader("sibling1")
  .newClassloader("sibling2")
  .newClassloader("the-child")
  .setParent("the-child", "the-parent", new Mask())
  .addSibling("the-child", "sibling1", new Mask())
  .addSibling("the-child", "sibling2", new Mask())
  .build();
```

#### Masks

A mask restricts access of a classloader to some resources and classes. By default there are no restrictions.
 
A mask is based on inclusion and exclusion patterns. Format is file path separated by slashes, for example "org/foo/Bar.class" or "org/foo/config.xml". Wildcard patterns are not supported. Directories must end with slash, for example "org/foo/" for excluding package org.foo and all its sub-packages.

Masks can be defined on parent-child and sibling relations.

```java
// all the resources/classes of classloader 'b' are visible from 'a', except org/foo/** resources.
ClassloaderBuilder builder = new ClassloaderBuilder();
Map<String, ClassLoader> classloaders = builder
  .newClassloader("a")
  .newClassloader("b")
  .addSibling("a", "b", new Mask().addExclusion("org/foo/"))
  .build();
```

When the same patterns are defined multiple times for each usage of a classloader, then the patterns can be declared once on the targetted classloader.

```java
// classloader 'a' exports only the resources/classes "org/a/api/" but not other internal classes. 
ClassloaderBuilder builder = new ClassloaderBuilder();
Map<String, ClassLoader> classloaders = builder
  .newClassloader("a")
  .setExportMask(new Mask().addInclusion("org/a/api/"))
  .newClassloader("b")
  .newClassloader("c")
  .addSibling("b", "a", new Mask())
  .addSibling("c", "a", new Mask())
  .build();
```

This is equivalent of:

```java
ClassloaderBuilder builder = new ClassloaderBuilder();
Map<String, ClassLoader> classloaders = builder
  .newClassloader("a")
  .newClassloader("b")
  .newClassloader("c")
  .addSibling("b", "a", new Mask().addInclusion("org/a/api/"))
  .addSibling("c", "a", new Mask().addInclusion("org/a/api/"))
  .build();
```

## License

    Copyright (C) 2015 SonarSource
    
    This program is free software; you can redistribute it and/or
    modify it under the terms of the GNU Lesser General Public
    License as published by the Free Software Foundation; either
    version 3 of the License, or (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
    Lesser General Public License for more details.

    You should have received a copy of the GNU Lesser General Public
    License along with this program; if not, write to the Free Software
    Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02

[classworlds]: http://classworlds.codehaus.org
[plexus]: https://github.com/sonatype/plexus-classworlds
