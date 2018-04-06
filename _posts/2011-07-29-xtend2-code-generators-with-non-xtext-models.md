---
layout: post
title: Xtend2 Code Generators with Non-Xtext Models
date: 2011-07-29 20:12:29.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- EMF
- Xtend
- Xtext
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---
In this blog post i want to show a simple example of how to use Xtend2 to generate code from Non-Xtext but EMF-based model.

![Xtend with EMF]({{ "/assets/img/blog/xtend-emf1.png" | absolute_url }})

Having a simple EMF Model i’ve created the genmodel + Model + Edit + Editor code.
Using the Editor i’ve created a bunch of .sample files and now want to
generate code using Xtend2

![Xtend EMF Models]({{ "/assets/img/blog/xtend-emf-models.png" | absolute_url }})

Xtend comes with an `IGenerator` interface that i implement in my `SampleGenerator` Xtend file

```
package sample
 
import org.eclipse.emf.ecore.resource.Resource
import org.eclipse.xtext.generator.IGenerator
import org.eclipse.xtext.generator.IFileSystemAccess
import org.eclipse.emf.ecore.EObject
 
class SampleGenerator implements IGenerator {
 
    override void doGenerate(Resource resource, IFileSystemAccess fsa) {
        for (EObject o : resource.contents) {
            o.compile(fsa)
        }
    }
 
    def dispatch void compile(Model m, IFileSystemAccess fsa) {
        for (e : m.elements) {
            e.compile(fsa)
        }
    }
 
    def compile(Element e, IFileSystemAccess fsa) {
        fsa.generateFile(e.name+".txt", '''
        this is element «e.name»
        ''')
    }
 
    def dispatch void compile(EObject m, IFileSystemAccess fsa) { }
 
}
``` 
The last step we need is a workflow that reads the model files and invokes the generator

First we need to create some java classes that exposes our .sample to the reader
(resourceseriveprovider) and
do some Guice Binding Stuff (Generator / ResourceSet ….)


```
package sample;
 
import org.eclipse.emf.ecore.resource.ResourceSet;
import org.eclipse.emf.ecore.resource.impl.ResourceSetImpl;
import org.eclipse.xtext.generator.IGenerator;
import org.eclipse.xtext.resource.generic.AbstractGenericResourceRuntimeModule;
 
public class SampleGeneratorModule extends AbstractGenericResourceRuntimeModule {
 
    @Override
    protected String getLanguageName() {
        return "sample.presentation.SampleEditorID";
    }
 
    @Override
    protected String getFileExtensions() {
        return "sample";
    }
 
    public Class<? extends IGenerator> bindIGenerator() {
        return SampleGenerator.class;
    }
 
    public Class<? extends ResourceSet> bindResourceSet() {
        return ResourceSetImpl.class;
    }
 
}
```


```
package sample;
 
import org.eclipse.xtext.ISetup;
 
import com.google.inject.Guice;
import com.google.inject.Injector;
 
public class SampleGeneratorSetup implements ISetup {
 
    @Override
    public Injector createInjectorAndDoEMFRegistration() {
        return Guice.createInjector(new SampleGeneratorModule());
    }
 
}
```


```
package sample;
 
import org.eclipse.xtext.resource.generic.AbstractGenericResourceSupport;
 
import com.google.inject.Module;
 
public class SampleGeneratorSupport extends AbstractGenericResourceSupport {
 
    @Override
    protected Module createGuiceModule() {
        return new SampleGeneratorModule();
    }
 
}
```

finally we wire this together in the workflow file
```
module sample.SampleGenerator
 
import org.eclipse.emf.mwe.utils.*
 
var targetDir = "src-gen"
var modelPath = "model"
 
Workflow {
 
    bean = StandaloneSetup {
        registerGeneratedEPackage = "sample.SamplePackage"
    }
 
    component = DirectoryCleaner {
        directory = targetDir
    }
 
    component = sample.SampleGeneratorSupport {}
 
    component = org.eclipse.xtext.mwe.Reader {
        path = modelPath
        register = sample.SampleGeneratorSetup {}
        loadResource = {
            slot = "model"
        }
    }
 
    component = org.eclipse.xtext.generator.GeneratorComponent {
        register = sample.SampleGeneratorSetup {}
        slot = 'model'
        outlet = {
            path = targetDir
        }
    }
}
```

running the workflow we get nice files generated

![Xtend EMF Models + Generated]({{ "/assets/img/blog/xtend-emf-result.png" | absolute_url }})