---
layout: post
title: Xtext Model Visualization with PlantUML
date: 2013-04-12 08:53:01.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags: []
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---
One of my Colleagues recently gave me a hint on PlantUML which is a nice tool to create Graphviz based UML diagrams from a textual input. This blogpost describes how to include PlantUML into Xtext to generate Visualizations from textual models on the fly.

Here is the DSL from

```
Domainmodel :
  elements += Type*
;
   
Type:
  DataType | Entity
;
   
DataType:
  'datatype' name = ID
;
  
Entity:
  'entity' name = ID ('extends' superType = [Entity])? '{'
     features += Feature*
  '}'
;
  
Feature:
  many?='many'? name = ID ':' type = [Type]
;
```

The target is to take an input model like

```
datatype String
 
entity A {
    many names : String
    c : C
}
 
entity B {
    something : String
    many myA : A
}
 
entity C {
     
} 
```

An generate a nice Diagram like

![Plant UML Sample]({{ "/assets/img/blog/test-mydsl.png" | absolute_url }})

To make the integration easy we generate the png using the existing Builder/Generator infrastructure

So here is the text input for PlantUML we need to generate

```
@startuml
class A {
    List<String> names
}
 
A o--  C : c
 
class B {
    String something
}
 
B o-- "*"  A : myA
 
class C {
}
 
 
@enduml
``` 

and here the generator that does the conversion and feeds Plantuml

```
class MyDslGenerator implements IGenerator {
 
    override void doGenerate(Resource resource, IFileSystemAccess fsa) {
        val filename = resource.URI.lastSegment
        for (dm : resource.contents.filter(typeof(Domainmodel))) {
            val plantUML = dm.toPlantUML.toString
            if (fsa instanceof IFileSystemAccessExtension3) {
                val out = new ByteArrayOutputStream()
                new SourceStringReader(plantUML).generateImage(out)
                (fsa as IFileSystemAccessExtension3).generateFile(filename + ".png",
                    new ByteArrayInputStream(out.toByteArray))
            } else {
                fsa.generateFile(filename + ".txt", plantUML)
            }
        }
    }
 
     
    def dispatch CharSequence toPlantUML(Domainmodel it) '''
    @startuml
    «FOR e : elements.filter(typeof(Entity))»
    «e.toPlantUML»
    «ENDFOR»
    @enduml
    '''
     
    def dispatch CharSequence toPlantUML(Entity it) '''
    class «name» {
        «FOR f : features.filter[type instanceof DataType]»
        «IF f.many»List<«f.type.name»>«ELSE»«f.type.name»«ENDIF» «f.name»
        «ENDFOR»
    }
     
    «FOR f : features.filter[type instanceof Entity]»
        «name» o-- «IF f.many»"*" «ENDIF» «f.type.name» : «f.name»
    «ENDFOR»
     
    ''' 
}
``` 

To get PlantUML into the Classpath we add the jar to the project
and add it via the Manifest.MF file

```
Bundle-ClassPath: .,
 lib/plantuml.jar
``` 