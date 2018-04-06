---
layout: post
title: Xtext 2.0 and UML
date: 2011-07-17 13:15:18.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
- UML
- IResourceServiceProvider
tags: []
author: Christian Dietrich
---
Consider you have some (Eclipse) UML2 models and now want to reference e.g. Classes from these model from your dsl. In this blog post i want to show a simple example what to do. We have to do two things: Provide an `IResourceServiceProvider` for `.uml` resources and reference the UML class from the grammar. So first we write the grammar. that is quite easy

```
grammar org.xtext.example.umldsl.UmlDsl with org.eclipse.xtext.common.Terminals
 
import "http://www.eclipse.org/uml2/3.0.0/UML" as uml
import "http://www.eclipse.org/emf/2002/Ecore" as ecore
 
generate umlDsl "http://www.xtext.org/example/umldsl/UmlDsl"
 
Model:
    elements+=Element*
;
 
Element:
    "element" name=ID "mapsTo" ref=[uml::Class|FQN]
;
 
FQN returns ecore::EString:
    ID ("." ID)*
;
```
Then we have to add some stuff to the Language workflow to get the stuff running

```
module org.xtext.example.umldsl.GenerateUmlDsl
 
import org.eclipse.emf.mwe.utils.*
import org.eclipse.xtext.generator.*
import org.eclipse.xtext.ui.generator.*
 
var grammarURI = "classpath:/org/xtext/example/umldsl/UmlDsl.xtext"
var file.extensions = "umldsl"
var projectName = "org.xtext.example.umldsl"
var runtimeProject = "../${projectName}"
 
Workflow {
    bean = StandaloneSetup {
        scanClassPath = true
        platformUri = "${runtimeProject}/.."
        uriMap = {
            from = "platform:/plugin/org.eclipse.emf.codegen.ecore/model/GenModel.genmodel"
            to = "platform:/resource/org.eclipse.emf.codegen.ecore/model/GenModel.genmodel"
        }
        uriMap = {
            from = "platform:/plugin/org.eclipse.emf.ecore/model/Ecore.genmodel"
            to = "platform:/resource/org.eclipse.emf.ecore/model/Ecore.genmodel"
        }
        uriMap = {
            from = "platform:/plugin/org.eclipse.uml2.codegen.ecore/model/GenModel.genmodel"
            to = "platform:/resource/org.eclipse.uml2.codegen.ecore/model/GenModel.genmodel"
        }
        uriMap = {
            from = "platform:/plugin/org.eclipse.uml2.uml/model/UML.genmodel"
            to = "platform:/resource/org.eclipse.uml2.uml/model/UML.genmodel"
        }
        uriMap = {
            from = "platform:/plugin/org.eclipse.emf.codegen.ecore/model/GenModel.ecore"
            to = "platform:/resource/org.eclipse.emf.codegen.ecore/model/GenModel.ecore"
        }
        uriMap = {
            from = "platform:/plugin/org.eclipse.emf.ecore/model/Ecore.ecore"
            to = "platform:/resource/org.eclipse.emf.ecore/model/Ecore.ecore"
        }
        uriMap = {
            from = "platform:/plugin/org.eclipse.uml2.codegen.ecore/model/GenModel.ecore"
            to = "platform:/resource/org.eclipse.uml2.codegen.ecore/model/GenModel.ecore"
        }
        uriMap = {
            from = "platform:/plugin/org.eclipse.uml2.uml/model/UML.ecore"
            to = "platform:/resource/org.eclipse.uml2.uml/model/UML.ecore"
        }
        //
        registerGeneratedEPackage = "org.eclipse.emf.ecore.EcorePackage"
        registerGeneratedEPackage = "org.eclipse.uml2.uml.UMLPackage"
        registerGeneratedEPackage = "org.eclipse.emf.codegen.ecore.genmodel.GenModelPackage"
        registerGeneratedEPackage = "org.eclipse.uml2.codegen.ecore.genmodel.GenModelPackage"
        registerGenModelFile = "platform:/resource/org.eclipse.emf.ecore/model/Ecore.genmodel"
        registerGenModelFile = "platform:/resource/org.eclipse.emf.codegen.ecore/model/GenModel.genmodel"
        registerGenModelFile = "platform:/resource/org.eclipse.uml2.uml/model/UML.genmodel"
        registerGenModelFile = "platform:/resource/org.eclipse.uml2.codegen.ecore/model/GenModel.genmodel"
 
    }
 
    ...
}
```
We add some extra deps to the manifest
```
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: org.xtext.example.umldsl
Bundle-Vendor: My Company
Bundle-Version: 1.0.0
Bundle-SymbolicName: org.xtext.example.umldsl; singleton:=true
Bundle-ActivationPolicy: lazy
Require-Bundle: org.eclipse.xtext;bundle-version="2.0.0";visibility:=reexport,
 org.apache.log4j;bundle-version="1.2.15";visibility:=reexport,
 org.apache.commons.logging;bundle-version="1.0.4";resolution:=optional;visibility:=reexport,
 org.eclipse.xtext.generator;resolution:=optional,
 org.eclipse.emf.codegen.ecore;resolution:=optional,
 org.eclipse.emf.mwe.utils;resolution:=optional,
 org.eclipse.emf.mwe2.launch;resolution:=optional,
 org.eclipse.uml2.uml;bundle-version="3.2.0",
 org.eclipse.xtext.util,
 org.eclipse.emf.ecore,
 org.eclipse.emf.common,
 org.antlr.runtime,
 org.eclipse.xtext.common.types,
 org.eclipse.uml2.codegen.ecore;bundle-version="1.7.0"
Import-Package: org.apache.log4j,
 org.apache.commons.logging,
 org.eclipse.xtext.xbase.lib,
 org.eclipse.xtext.xtend2.lib
Bundle-RequiredExecutionEnvironment: J2SE-1.5
Export-Package: org.xtext.example.umldsl,
 org.xtext.example.umldsl.services,
 org.xtext.example.umldsl.umlDsl,
 org.xtext.example.umldsl.umlDsl.impl,
 org.xtext.example.umldsl.umlDsl.util,
 org.xtext.example.umldsl.serializer,
 org.xtext.example.umldsl.parser.antlr,
 org.xtext.example.umldsl.parser.antlr.internal,
 org.xtext.example.umldsl.validation
```
and generate the Language. If we now run a runtime application and create a .uml file and a .umlmodel file we see nothing since the uml stuff is not yet referenceable 

![UML Not Working]({{ "/assets/img/blog/xtext-uml-1.png" | absolute_url }})

So the second with we create is a `IResourceServiceProvider` . Therefore we take the plugins `org.eclipse.xtext.ecore` and `org.eclipse.xtext.ui.ecore? , that do the same for `.ecore` files, as inspiration. So we create the plugins `org.eclipse.xtext.uml` and `org.eclipse.xtext.ui.uml` with following content: 

![UML Plugin]({{ "/assets/img/blog/uml-plugin.png" | absolute_url }})

The`UmlResourceDescriptionStrategy` customizes the creation of `IEObjectDescription`s. This is the stuff that Xtext puts into the index and maps a Name to an EObject or Proxy. In our case we simply subclass from the default and do no further customizations.

```
package org.eclipse.xtext.uml;
 
import org.eclipse.xtext.resource.impl.DefaultResourceDescriptionStrategy;
 
public class UmlResourceDescriptionStrategy extends DefaultResourceDescriptionStrategy {
 
}
```
The `UmlQualifiedNameProvider` gives our UML Stuff a fully qualified name – we take the defaults here too
```
package org.eclipse.xtext.uml;
 
import org.eclipse.xtext.naming.DefaultDeclarativeQualifiedNameProvider;
 
public class UmlQualifiedNameProvider extends DefaultDeclarativeQualifiedNameProvider {
 
}
```
Then we have to create a Guice Module to Glue the stuff
```
package org.eclipse.xtext.uml;
 
import org.eclipse.xtext.naming.IQualifiedNameProvider;
import org.eclipse.xtext.resource.IDefaultResourceDescriptionStrategy;
import org.eclipse.xtext.resource.generic.AbstractGenericResourceRuntimeModule;
 
public class UmlRuntimeModule extends AbstractGenericResourceRuntimeModule {
 
    @Override
    protected String getLanguageName() {
        return "org.eclipse.uml2.uml.editor.presentation.UMLEditorID";
    }
 
    @Override
    protected String getFileExtensions() {
        return "uml";
    }
 
    public Class<? extends IDefaultResourceDescriptionStrategy> bindIDefaultResourceDescriptionStrategy() {
        return UmlResourceDescriptionStrategy.class;
    }
 
    @Override
    public Class<? extends IQualifiedNameProvider> bindIQualifiedNameProvider() {
        return UmlQualifiedNameProvider.class;
    }
 
}
```
If we want to use this stuff from an mwe(2) workflow we have to create a Support-Class for this too

```
package org.eclipse.xtext.uml;
 
import org.eclipse.xtext.resource.generic.AbstractGenericResourceSupport;
 
import com.google.inject.Module;
 
public class UmlSupport extends AbstractGenericResourceSupport {
 
    @Override
    protected Module createGuiceModule() {
        return new UmlRuntimeModule();
    }
 
}
```
Finally we add some stuff to the manifest and we’re done with the runtime stuf
```
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: Uml
Bundle-SymbolicName: org.eclipse.xtext.uml
Bundle-Version: 1.0.0.qualifier
Bundle-RequiredExecutionEnvironment: JavaSE-1.6
Require-Bundle: org.eclipse.xtext;bundle-version="2.0.0",
 org.eclipse.uml2.uml;bundle-version="3.2.0"
Export-Package: org.eclipse.xtext.uml
```

![UML UI Plugin]({{ "/assets/img/blog/uml-ui-plugin.png" | absolute_url }})

Then we have to do some stuff at the ui side too. We create some glue code (`Activator`, `ExecutableExtensionFactory` (to be able to use guice in the `plugin.xml`) and an `UiModule`)

```
/*******************************************************************************
 * Copyright (c) 2010 itemis AG (http://www.itemis.eu) and others.
 * All rights reserved. This program and the accompanying materials
 * are made available under the terms of the Eclipse Public License v1.0
 * which accompanies this distribution, and is available at
 * http://www.eclipse.org/legal/epl-v10.html
 *******************************************************************************/
package org.eclipse.xtext.ui.uml;
 
import org.apache.log4j.Logger;
import org.eclipse.ui.plugin.AbstractUIPlugin;
import org.eclipse.xtext.ui.shared.SharedStateModule;
import org.eclipse.xtext.uml.UmlRuntimeModule;
import org.osgi.framework.BundleContext;
 
import com.google.inject.Guice;
import com.google.inject.Injector;
import com.google.inject.util.Modules;
 
/**
 * The activator class controls the plug-in life cycle
 */
public class Activator extends AbstractUIPlugin {
 
    private static final Logger logger = Logger.getLogger(Activator.class);
 
    // The plug-in ID
    public static final String PLUGIN_ID = "org.eclipse.xtext.ui.uml"; //$NON-NLS-1$
 
    // The shared instance
    private static Activator plugin;
 
    private Injector injector;
 
    /**
     * The constructor
     */
    public Activator() {
    }
 
    public Injector getInjector() {
        return injector;
    }
 
    private void initializeEcoreInjector() {
        injector = Guice.createInjector(
                Modules.override(Modules.override(new UmlRuntimeModule())
                .with(new UmlUiModule(plugin)))
                .with(new SharedStateModule()));
    }
 
    @Override
    public void start(BundleContext context) throws Exception {
        super.start(context);
        plugin = this;
        try {
            initializeEcoreInjector();
        } catch(Exception e) {
            logger.error(e.getMessage(), e);
            throw e;
        }
    }
 
    @Override
    public void stop(BundleContext context) throws Exception {
        plugin = null;
        injector = null;
        super.stop(context);
    }
 
    /**
     * Returns the shared instance
     *
     * @return the shared instance
     */
    public static Activator getDefault() {
        return plugin;
    }
 
}
```

```
/*******************************************************************************
 * Copyright (c) 2010 itemis AG (http://www.itemis.eu) and others.
 * All rights reserved. This program and the accompanying materials
 * are made available under the terms of the Eclipse Public License v1.0
 * which accompanies this distribution, and is available at
 * http://www.eclipse.org/legal/epl-v10.html
 *******************************************************************************/
package org.eclipse.xtext.ui.uml;
 
import org.eclipse.ui.plugin.AbstractUIPlugin;
import org.eclipse.xtext.ui.LanguageSpecific;
import org.eclipse.xtext.ui.editor.IURIEditorOpener;
import org.eclipse.xtext.ui.resource.generic.EmfUiModule;
 
public class UmlUiModule extends EmfUiModule {
 
    public UmlUiModule(AbstractUIPlugin plugin) {
        super(plugin);
    }
 
    @Override
    public void configureLanguageSpecificURIEditorOpener(com.google.inject.Binder binder) {
        binder.bind(IURIEditorOpener.class).annotatedWith(LanguageSpecific.class).to(UmlEditorOpener.class);
    }
 
}
``` 


```
/*******************************************************************************
 * Copyright (c) 2010 itemis AG (http://www.itemis.eu) and others.
 * All rights reserved. This program and the accompanying materials
 * are made available under the terms of the Eclipse Public License v1.0
 * which accompanies this distribution, and is available at
 * http://www.eclipse.org/legal/epl-v10.html
 *******************************************************************************/
package org.eclipse.xtext.ui.uml;
 
import org.eclipse.xtext.ui.guice.AbstractGuiceAwareExecutableExtensionFactory;
import org.osgi.framework.Bundle;
 
import com.google.inject.Injector;
 
public class ExecutableExtensionFactory extends AbstractGuiceAwareExecutableExtensionFactory {
 
    @Override
    protected Bundle getBundle() {
        return Activator.getDefault().getBundle();
    }
 
    @Override
    protected Injector getInjector() {
        return Activator.getDefault().getInjector();
    }
 
}
``` 

Of course we want to UML Editor to open smoothly if we click on an element referenced from UML too so we create an `LanguageSpecificURIEditorOpener` too


```
/*******************************************************************************
 * Copyright (c) 2010 itemis AG (http://www.itemis.eu) and others.
 * All rights reserved. This program and the accompanying materials
 * are made available under the terms of the Eclipse Public License v1.0
 * which accompanies this distribution, and is available at
 * http://www.eclipse.org/legal/epl-v10.html
 *******************************************************************************/
package org.eclipse.xtext.ui.uml;
 
import java.util.Collections;
 
import org.eclipse.emf.common.util.URI;
import org.eclipse.emf.ecore.EObject;
import org.eclipse.emf.ecore.EReference;
import org.eclipse.ui.IEditorPart;
import org.eclipse.xtext.ui.editor.LanguageSpecificURIEditorOpener;
import org.eclipse.uml2.uml.editor.presentation.UMLEditor;
 
public class UmlEditorOpener extends LanguageSpecificURIEditorOpener {
 
    @Override
    protected void selectAndReveal(IEditorPart openEditor, URI uri,
            EReference crossReference, int indexInList, boolean select) {
        UMLEditor umlEditor = (UMLEditor) openEditor.getAdapter(UMLEditor.class);
        if (umlEditor != null) {
            EObject eObject = umlEditor.getEditingDomain().getResourceSet().getEObject(uri, true);
            umlEditor.setSelectionToViewer(Collections.singletonList(eObject));
        }
    }
 
}
``` 

we add some deps to the manifest

```
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
Bundle-Name: Uml
Bundle-SymbolicName: org.eclipse.xtext.ui.uml;singleton:=true
Bundle-Version: 1.0.0.qualifier
Bundle-RequiredExecutionEnvironment: JavaSE-1.6
Require-Bundle: org.eclipse.uml2.uml.editor;bundle-version="3.1.100",
 org.eclipse.xtext.uml;bundle-version="1.0.0",
 org.eclipse.xtext.ui;bundle-version="2.0.0",
 org.eclipse.xtext.ui.shared;bundle-version="2.0.0"
Import-Package: org.apache.log4j;version="1.2.15"
Bundle-Activator: org.eclipse.xtext.ui.uml.Activator
Bundle-ActivationPolicy: lazy
``` 
And finally register our resource service provider to the extension point Xtext offers for that.

```
<?xml version="1.0" encoding="UTF-8"?>
<?eclipse version="3.4"?>
<plugin>
     
    <extension
          point="org.eclipse.xtext.extension_resourceServiceProvider">
        <resourceServiceProvider
              class="org.eclipse.xtext.ui.uml.ExecutableExtensionFactory:org.eclipse.xtext.ui.resource.generic.EmfResourceUIServiceProvider"
              uriExtension="uml">
        </resourceServiceProvider>
    </extension>
  
</plugin>
``` 

We restart our runtime application, and TATATATA: it works 😉

![UML Working]({{ "/assets/img/blog/uml-xtext-working.png" | absolute_url }})

the source code can be found at [https://github.com/cdietrich/xtext-uml-example](https://github.com/cdietrich/xtext-uml-example)