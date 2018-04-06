---
layout: post
title: Use of EcoreGenerator to customize EMFs generated Java Classes
date: 2013-07-26 08:15:18.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Xtext
- Ecore
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---
This is a blog post on the usage of the org.eclipse.emf.mwe2.ecore.EcoreGenerator worflow component.
It is an alternative to EMFs JMerge Based approach to customize generated Java Classes from an ecore model.

So let us asume we have the following Ecore file
(/sample/model/sample.ecore)

```
<?xml version="1.0" encoding="UTF-8"?>
<ecore:EPackage xmi:version="2.0" xmlns:xmi="http://www.omg.org/XMI" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:ecore="http://www.eclipse.org/emf/2002/Ecore" name="sample" nsURI="http://www.eclipse.org/xtext/ecore/sample" nsPrefix="sample">
  <eClassifiers xsi:type="ecore:EClass" name="Person">
    <eOperations name="getFullname" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="firstname" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
    <eStructuralFeatures xsi:type="ecore:EAttribute" name="lastname" eType="ecore:EDataType http://www.eclipse.org/emf/2002/Ecore#//EString"/>
  </eClassifiers>
</ecore:EPackage>
```
with the following genmodel
(/sample/model/sample.genmodel)
```
<?xml version="1.0" encoding="UTF-8"?>
<genmodel:GenModel xmi:version="2.0" xmlns:xmi="http://www.omg.org/XMI" xmlns:ecore="http://www.eclipse.org/emf/2002/Ecore"
    xmlns:genmodel="http://www.eclipse.org/emf/2002/GenModel" modelDirectory="/sample/src-gen" modelPluginID="sample" modelName="Sample"
    importerID="org.eclipse.emf.importer.ecore" complianceLevel="6.0" copyrightFields="false">
  <foreignModel>sample.ecore</foreignModel>
  <genPackages prefix="Sample" disposableProviderFactory="true" ecorePackage="sample.ecore#/">
    <genClasses ecoreClass="sample.ecore#//Person">
      <genFeatures createChild="false" ecoreFeature="ecore:EAttribute sample.ecore#//Person/firstname"/>
      <genFeatures createChild="false" ecoreFeature="ecore:EAttribute sample.ecore#//Person/lastname"/>
      <genOperations ecoreOperation="sample.ecore#//Person/getFullname"/>
    </genClasses>
  </genPackages>
</genmodel:GenModel>
```
I have changed the modelDirectory="/sample/src-gen"

Now we want to implement the getFullname() method.
we would usually implement the code in PersonImpl and change the javadoc to @generated NOT or remove the @generated.
(and checkin the generated code as well)

But let us try another approach

so let us create following class

(/sample/src/sample/impl/PersonImplCustom.java)
```
package sample.impl;
 
import sample.impl.PersonImpl;
 
public class PersonImplCustom extends PersonImpl {
     
    @Override
    public String getFullname() {
        return getFirstname() + " " + getLastname();
    }
 
}
```
and workflow

(/sample/src/test.mwe2)
```
module test
 
Workflow {
     
    bean = org.eclipse.emf.mwe.utils.StandaloneSetup {
        platformUri = ".."
    }
     
    component = org.eclipse.emf.mwe.utils.DirectoryCleaner {
        directory = "src-gen"
    }
     
    component = org.eclipse.emf.mwe2.ecore.EcoreGenerator {
        generateCustomClasses = false
        genModel = "platform:/resource/sample/model/sample.genmodel"
        srcPath = "platform:/resource/sample/src"   
    }
     
}
```
here is the Manifest for the deps
(/sample/META-INF/MANIFEST.MF)
```
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: sample
Bundle-SymbolicName: sample; singleton:=true
Bundle-Version: 0.1.0
Require-Bundle: org.eclipse.emf.mwe2.lib,
 org.eclipse.emf.mwe2.launch,
 org.apache.commons.logging,
 org.apache.commons.lang,
 org.eclipse.xtext.xbase.lib,
 org.eclipse.xtext,
 org.eclipse.xtext.generator,
 org.apache.commons.logging,
 org.eclipse.ui.workbench,
 org.eclipse.core.runtime,
 org.eclipse.core.commands,
 org.eclipse.xtext.ui,
 org.eclipse.core.expressions,
 org.apache.log4j
Export-Package: sample.impl,
 sample.util
```
If we run the workflow emf is configured to take PersonImplCustom instead of PersonImpl so that the following will work nicely
```
public class Test {
     
    public static void main(String[] args) {
        Person p = SampleFactory.eINSTANCE.createPerson();
        p.setFirstname("Christian");
        p.setLastname("Dietrich");
        System.out.println(p.getFullname());
    }
 
}
```