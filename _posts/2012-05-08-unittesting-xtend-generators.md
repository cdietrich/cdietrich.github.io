---
layout: post
title: Unittesting Xtend Generators
date: 2012-05-08 19:11:15.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
- Generator
- Tests
tags: []
meta:
  _edit_last: '25129546'
author: hristian Dietrich
---
Xtext offers nice Support for Unit Tests. But how to test a Xtend based Generator? This blogpost describes a simple approach for such a Test.

So let us take Xtext’s Hello World grammar as Starting point

```
Model:
    greetings+=Greeting*;
     
Greeting:
    'Hello' name=ID '!';
```
And following simple Generator
```
package org.xtext.example.mydsl.generator
 
import org.eclipse.emf.ecore.resource.Resource
import org.eclipse.xtext.generator.IFileSystemAccess
import org.eclipse.xtext.generator.IGenerator
import org.xtext.example.mydsl.myDsl.Greeting
 
class MyDslGenerator implements IGenerator {
     
    override void doGenerate(Resource resource, IFileSystemAccess fsa) {
        for (g : resource.allContents.toIterable.filter(typeof(Greeting))) {
            fsa.generateFile(g.name+".java", 
            '''
            public class «g.name» {
                 
            }
            ''')
        }
    }
}
```
And here the Test
```
import org.junit.Test
import org.junit.runner.RunWith
import org.eclipse.xtext.junit4.XtextRunner
import org.eclipse.xtext.junit4.InjectWith
import org.xtext.example.mydsl.MyDslInjectorProvider
import org.eclipse.xtext.generator.IGenerator
import com.google.inject.Inject
import org.eclipse.xtext.junit4.util.ParseHelper
import org.xtext.example.mydsl.myDsl.Model
import org.eclipse.xtext.generator.InMemoryFileSystemAccess
 
import static org.junit.Assert.*
import org.eclipse.xtext.generator.IFileSystemAccess
 
@RunWith(typeof(XtextRunner))
@InjectWith(typeof(MyDslInjectorProvider))
class GeneratorTest {
     
    @Inject IGenerator underTest
    @Inject ParseHelper<Model> parseHelper 
     
    @Test
    def test() {
        val model = parseHelper.parse('''
        Hello Alice!
        Hello Bob!
        ''')
        val fsa = new InMemoryFileSystemAccess()
        underTest.doGenerate(model.eResource, fsa)
        println(fsa.files)
        assertEquals(2,fsa.files.size)
        assertTrue(fsa.files.containsKey(IFileSystemAccess::DEFAULT_OUTPUT+"Alice.java"))
        assertEquals(
            '''
            public class Alice {
                 
            }
            '''.toString, fsa.files.get(IFileSystemAccess::DEFAULT_OUTPUT+"Alice.java").toString
        )
        assertTrue(fsa.files.containsKey(IFileSystemAccess::DEFAULT_OUTPUT+"Bob.java"))
        assertEquals(
            '''
            public class Bob {
                 
            }
            '''.toString, fsa.files.get(IFileSystemAccess::DEFAULT_OUTPUT+"Bob.java").toString)
         
    }
     
}
```
But how does that work?

Xtext offers a specific org.junit.runner.Runner. For Junit4 it is `org.junit.runner.Runner`. This Runner allows in combination with a
`org.eclipse.xtext.junit4.IInjectorProvide` language specific injections within the test.
Since we have `fragment = junit.Junit4Fragment {}` in our workflow
Xtext already Generated the Class `org.xtext.example.mydsl.MyDslInjectorProvider`.
If we would not use Xtext at all we would have to create such a InjectorProvider manually.

To wire these things up we annotate your Test with `@RunWith(typeof(XtextRunner))` and `@InjectWith(typeof(MyDslInjectorProvider))`

Now we can write our Test. This Basically consists of 3 steps
1. read a model
1. call the Generator
1. Capture the Result

We solve Step (1) using Xtext’s `org.eclipse.xtext.junit4.util.ParseHelper` and Step (3) by using a special kind of `IFileSystemAccess` that keeps the files InMemory and does not write them to the disk.

I hope this gives you a start writing you Xtext/Xtend Generator Tests.