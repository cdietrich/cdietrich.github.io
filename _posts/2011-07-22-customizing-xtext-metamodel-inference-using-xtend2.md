---
layout: post
title: Customizing Xtext Metamodel Inference using Xtend2
date: 2011-07-22 22:11:29.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Xtext
- Inference
- Metamodel
- PostProcessor
- Xtend
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---
Xtext has nice metamodel inference capabilities. But sometimes you have to do some customizations to the generated ecore metamodel, e.g. adding a derived operation. You have basically two options: (1) move to a manually maintained metamodel (2) use Customized Post Processing as described here [http://www.eclipse.org/Xtext/documentation/2_0_0/020-grammar-language.php#customPostProcessing](http://www.eclipse.org/Xtext/documentation/2_0_0/020-grammar-language.php#customPostProcessing)

The second possibility uses good old Xpand/Xtend1 extensions to do the postprocessing. But what if i want to use Xtend2 for that? A very simple solution i’d like to show in this post.

So lets start with the gramar

```
grammar org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals
 
generate myDsl "http://www.xtext.org/example/mydsl/MyDsl"
 
Model:
    persons+=Person*
;
 
Person:
    "person" firstname=STRING lastname=STRING
;
```

We now want to add a getFullname Operation to our person.
Xtext offers the Interface `IXtext2EcorePostProcessor` for the postprocessing.
So we write a Xtend class for that

```
package org.xtext.example.mydsl
 
import org.eclipse.xtext.xtext.ecoreInference.IXtext2EcorePostProcessor
import org.eclipse.xtext.GeneratedMetamodel
import org.eclipse.emf.ecore.EPackage
import org.eclipse.emf.ecore.EClassifier
import org.eclipse.emf.ecore.EClass
import org.eclipse.emf.ecore.EcoreFactory
import org.eclipse.emf.ecore.EcorePackage
import org.eclipse.emf.ecore.EcorePackage.Literals
import org.eclipse.emf.codegen.ecore.genmodel.GenModelPackage
import org.eclipse.emf.common.util.BasicEMap$Entry
import org.eclipse.emf.ecore.impl.EStringToStringMapEntryImpl
 
class MyXtext2EcorePostProcessor implements IXtext2EcorePostProcessor {
     
    override void process(GeneratedMetamodel metamodel) {
        metamodel.EPackage.process
    }
     
    def process(EPackage p) {
        for (c : p.EClassifiers.filter(typeof(EClass))) {
            if (c.name == "Person") {
                c.handle
            }
        }
    }
     
    def handle (EClass c) {
        val op = EcoreFactory::eINSTANCE.createEOperation
        op.name = "getFullName"
        op.EType = EcorePackage::eINSTANCE.EString
        val body = EcoreFactory::eINSTANCE.createEAnnotation
        body.source = GenModelPackage::eNS_URI
        val map = EcoreFactory::eINSTANCE.create(EcorePackage::eINSTANCE.getEStringToStringMapEntry()) as BasicEMap$Entry<String,String>
        map.key = "body"
        map.value = "return getFirstname() + \" \" + getLastname();"
        body.details.add(map)
        op.EAnnotations += body
        c.EOperations += op
    }
     
}
```
The last Problem left is how to make the Generator use this class.
Xtext does not offer a explicit place to change the `IXtext2EcorePostProcessor`.
Its default Implementation is bound in the `XtextRuntimeModule`,
that is instantiated in the `org.eclipse.xtext.generator.Generator`
class. so we have to subclass the Generator to get it changed

```
package org.xtext.example.mydsl;
 
import org.eclipse.xtext.XtextRuntimeModule;
import org.eclipse.xtext.XtextStandaloneSetup;
import org.eclipse.xtext.generator.Generator;
import org.eclipse.xtext.xtext.ecoreInference.IXtext2EcorePostProcessor;
 
import com.google.inject.Guice;
import com.google.inject.Injector;
 
@SuppressWarnings("restriction")
public class ExtendedGenerator extends Generator {
     
    public ExtendedGenerator() {
        new XtextStandaloneSetup() {
            @Override
            public Injector createInjector() {
                return Guice.createInjector(new XtextRuntimeModule() {
                    @Override
                    public Class<? extends IXtext2EcorePostProcessor> bindIXtext2EcorePostProcessor() {
                        return MyXtext2EcorePostProcessor.class;
                    }
                });
            }
        }.createInjectorAndDoEMFRegistration();
    }
 
}
```

finally we use the in the Generator Workflow

```
Workflow {
    bean = StandaloneSetup {
        scanClassPath = true
        platformUri = "${runtimeProject}/.."
    }
 
    component = DirectoryCleaner {
        directory = "${runtimeProject}/src-gen"
    }
 
    component = DirectoryCleaner {
        directory = "${runtimeProject}.ui/src-gen"
    }
 
    component = ExtendedGenerator {
        pathRtProject = runtimeProject
        pathUiProject = "${runtimeProject}.ui"
        pathTestProject = "${runtimeProject}.tests"
        projectNameRt = projectName
        projectNameUi = "${projectName}.ui"
        language = {
....
```
as a result our person has the getFullname Operation
```
public interface Person extends EObject
{
 
  String getFirstname();
 
 
  void setFirstname(String value);
 
 
  String getLastname();
 
 
  void setLastname(String value);
 
 
  String getFullName();
 
} // Person
```

```
public class PersonImpl extends MinimalEObjectImpl.Container implements Person
{
  
  /**
   * <!-- begin-user-doc -->
   * <!-- end-user-doc -->
   * @generated
   */
  public String getFullName()
  {
    return getFirstname() + " " + getLastname();
  }
 
 
 
} //PersonImpl
```