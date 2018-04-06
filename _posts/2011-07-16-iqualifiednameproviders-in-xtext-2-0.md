---
layout: post
title: IQualifiedNameProviders in Xtext 2.0
date: 2011-07-16 19:39:41.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- IQualifiedNameProvider
- Xtext
author: Christian Dietrich
---
Xtext 2.0 come with a change to the IQualifiedNameProvider interface. This interface is used to calculate a name for EObjects. The Name is used in Xtext’s index, for cross referencing and much more.

```
public interface IQualifiedNameProvider extends Function<EObject, QualifiedName> {
 
    QualifiedName getFullyQualifiedName(EObject obj);
 
}
```

Here we see the API change: There is a rename of the `getQualifiedName` method to `getFullyQualifedName`. In Xtext 1.0.x the qualified name was a simple String, now it is a wrapper class that holds the segements of the qualified name.

There are two default implementations for a `IQualifiedNameProvider`: `SimpleNameProvider` and `DefaultDeclarativeQualifiedNameProvider`. Consider we have a grammar like

```
grammar org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals
 
generate myDsl "http://www.xtext.org/example/mydsl/MyDsl"
 
Package:
    "package" name=ID "{"
        elements+=Element*
    "}"
;
 
Element:
    "element" name=ID
;
```
and a sample model like
```
package TestPackage {
    element A
    element B
}
```
Then with `SimpleNameProvider` we would have following qualified names.
* `TestPackage` for the package
* `A` and `B` for the elements

And with `DefaultDeclarativeQualifiedNameProvider` we would have following qualified names.
* `TestPackage` for the package
* `TestPackage.A` and `TestPackage.B` for the elements

Both Providers take the name EAttribute of our Package and Element to do the calculation. The `DefaultDeclarativeQualifiedNameProvider` uses the Elements parents qualified name too. (a fully qualified name ;-))
But it won’t work e.g. if our grammar would look like
```
grammar org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals
 
generate myDsl "http://www.xtext.org/example/mydsl/MyDsl"
 
Package:
    "package" name=ID "{"
        elements+=Element*
    "}"
;
 
Element:
    "element" id=ID
;
```
with `Element` having and id and not a name. We can easily change this by creating and binding our own `IQualifiedNameProvider` e.g. by extending `DefaultDeclarativeQualifiedNameProvider`
```
package org.xtext.example.mydsl;
 
import org.eclipse.xtext.naming.DefaultDeclarativeQualifiedNameProvider;
import org.eclipse.xtext.naming.QualifiedName;
import org.xtext.example.mydsl.myDsl.Element;
import org.xtext.example.mydsl.myDsl.Package;
 
public class MyDslQNP extends DefaultDeclarativeQualifiedNameProvider{
 
    QualifiedName qualifiedName(Element e) {
        Package p = (Package) e.eContainer();
        return QualifiedName.create(p.getName(), e.getId());
    }
 
}
```
We simply write a method qualifiedName that is called from the polymorpthic dispatcher when calculating the name of an Element
```
package org.xtext.example.mydsl;
 
import org.eclipse.xtext.naming.IQualifiedNameProvider;
 
/**
 * Use this class to register components to be used at runtime / without the Equinox extension registry.
 */
public class MyDslRuntimeModule extends org.xtext.example.mydsl.AbstractMyDslRuntimeModule {
 
    @Override
    public Class<? extends IQualifiedNameProvider> bindIQualifiedNameProvider() {
        return MyDslQNP.class;
    }
 
}

```