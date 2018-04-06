---
layout: post
title: 'Xtext: Calling the Generator from a Context Menu'
date: 2011-10-15 08:26:19.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Action
- Command
- Context Menu
- Generator
- Handler
- Xtend
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---
Xtext offers the user to implement an Xtend Class that implements the `org.eclipse.xtext.generator.IGenerator` Interface. By Default this Generator is called by the Builder through a `org.eclipse.xtext.builder.IXtextBuilderParticipant` when saving the file. But what to do if I want to call the Generator explicitely through a context menu entry on the file as shown in the screenshot below? This will be shown in the Following example.

![Xtext Generate from Context Menu]({{ "/assets/img/blog/generate-context-menu.png" | absolute_url }})

### Grammar & IGenerator ###

We first start with writing the Grammar and after generating the language we implement the Generator Stub Xtext created for us.

```
grammar org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals
 
generate myDsl "http://www.xtext.org/example/mydsl/MyDsl"
 
Model:
    greetings+=Greeting*;
     
Greeting:
    'Hello' name=ID ('from' from=[Greeting])?'!';
```


```
package org.xtext.example.mydsl.generator
 
import org.eclipse.emf.ecore.resource.Resource
import org.eclipse.xtext.generator.IGenerator
import org.eclipse.xtext.generator.IFileSystemAccess
import org.xtext.example.mydsl.myDsl.Greeting
 
import static extension org.eclipse.xtext.xtend2.lib.ResourceExtensions.*
 
class MyDslGenerator implements IGenerator {
     
    override void doGenerate(Resource resource, IFileSystemAccess fsa) {
        for (g : resource.allContentsIterable.filter(typeof(Greeting))) {
            fsa.generateFile(g.name+".txt",'''
                Hello «g.name» «IF g.from != null»from «g.from.name»«ENDIF»!
            ''')
        }
    }
}
```

### Disabling the IBuilderParticipant ###

Then we go to the UI Projects `plugin.xml` and disable the default registration of an `IXtextBuilderParticipant`

```
<!--
   <extension
         point="org.eclipse.xtext.builder.participant">
      <participant
            class="org.xtext.example.mydsl.ui.MyDslExecutableExtensionFactory:org.eclipse.xtext.builder.IXtextBuilderParticipant">
      </participant>
   </extension>
-->
```

### Setting up the Context Menu ###

Then we setup the context menu with Eclipse means

```
<extension
        point="org.eclipse.ui.handlers">
     <handler
           class="org.xtext.example.mydsl.ui.MyDslExecutableExtensionFactory:org.xtext.example.mydsl.ui.handler.GenerationHandler"
           commandId="org.xtext.example.mydsl.ui.handler.GenerationCommand">
     </handler>
      
  </extension>
   
  <extension
        point="org.eclipse.ui.commands">
        <command name="Generate Code"
              id="org.xtext.example.mydsl.ui.handler.GenerationCommand">
        </command>
  </extension>
   
  <extension point="org.eclipse.ui.menus">
    <menuContribution locationURI="popup:org.eclipse.jdt.ui.PackageExplorer">
        <command
            commandId="org.xtext.example.mydsl.ui.handler.GenerationCommand"
            style="push">
            <visibleWhen
                  checkEnabled="false">
                  <iterate>
       <adapt type="org.eclipse.core.resources.IResource">
          <test property="org.eclipse.core.resources.name" 
                value="*.mydsl"/>
       </adapt>
    </iterate>
            </visibleWhen>
        </command>
    </menuContribution>
    </extension>
```

### Implementing the Handler / Calling the Generator ###
The last thing we have to do is to call the IGenerator from the handler class.
this could look like

```
public class GenerationHandler extends AbstractHandler implements IHandler {
     
    @Inject
    private IGenerator generator;
 
    @Inject
    private Provider<EclipseResourceFileSystemAccess> fileAccessProvider;
     
    @Inject
    IResourceDescriptions resourceDescriptions;
     
    @Inject
    IResourceSetProvider resourceSetProvider;
     
    @Override
    public Object execute(ExecutionEvent event) throws ExecutionException {
         
        ISelection selection = HandlerUtil.getCurrentSelection(event);
        if (selection instanceof IStructuredSelection) {
            IStructuredSelection structuredSelection = (IStructuredSelection) selection;
            Object firstElement = structuredSelection.getFirstElement();
            if (firstElement instanceof IFile) {
                IFile file = (IFile) firstElement;
                IProject project = file.getProject();
                IFolder srcGenFolder = project.getFolder("src-gen");
                if (!srcGenFolder.exists()) {
                    try {
                        srcGenFolder.create(true, true,
                                new NullProgressMonitor());
                    } catch (CoreException e) {
                        return null;
                    }
                }
 
                final EclipseResourceFileSystemAccess fsa = fileAccessProvider.get();
                fsa.setOutputPath(srcGenFolder.getFullPath().toString());
                 
                URI uri = URI.createPlatformResourceURI(file.getFullPath().toString(), true);
                ResourceSet rs = resourceSetProvider.get(project);
                Resource r = rs.getResource(uri, true);
                generator.doGenerate(r, fsa);
                 
            }
        }
        return null;
    }
 
    @Override
    public boolean isEnabled() {
        return true;
    }
 
}
```

### Give it a try ###

We start a runtime app, create a bunch of model files that refernce each other,
try the genenerator – and hey – it works.

![Result]({{ "/assets/img/blog/result.png" | absolute_url }})

### Editor Context Menu ###

And how if i want to call the generator from the context menu of the open editor

![Context Menu Editor]({{ "/assets/img/blog/context-menu-on-editor.png" | absolute_url }})

I register the command for the editor and write a Handler like this one

```
<extension point="org.eclipse.ui.menus">
    <menuContribution locationURI="popup:#TextEditorContext?after=additions">
        <command
            commandId="org.xtext.example.mydsl.ui.handler.GenerationCommand"
            style="push">
            <visibleWhen
                      checkEnabled="false">
                   <reference
                         definitionId="org.xtext.example.mydsl.MyDsl.Editor.opened">
                   </reference>
                </visibleWhen>
        </command>
    </menuContribution>
</extension>
```

```
public class GenerationHandler extends AbstractHandler implements IHandler {
     
    @Inject
    private IGenerator generator;
 
    @Inject
    private Provider<EclipseResourceFileSystemAccess> fileAccessProvider;
     
    @Inject
    IResourceDescriptions resourceDescriptions;
     
    @Inject
    IResourceSetProvider resourceSetProvider;
     
    @Override
    public Object execute(ExecutionEvent event) throws ExecutionException {
         
        IEditorPart activeEditor = HandlerUtil.getActiveEditor(event);
        IFile file = (IFile) activeEditor.getEditorInput().getAdapter(IFile.class);
        if (file != null) {
            IProject project = file.getProject();
            IFolder srcGenFolder = project.getFolder("src-gen");
            if (!srcGenFolder.exists()) {
                try {
                    srcGenFolder.create(true, true,
                            new NullProgressMonitor());
                } catch (CoreException e) {
                    return null;
                }
            }
     
            final EclipseResourceFileSystemAccess fsa = fileAccessProvider.get();
            fsa.setOutputPath(srcGenFolder.getFullPath().toString());
             
             
            if (activeEditor instanceof XtextEditor) {
                ((XtextEditor)activeEditor).getDocument().readOnly(new IUnitOfWork<Boolean, XtextResource>() {
                 
                    @Override
                    public Boolean exec(XtextResource state)
                            throws Exception {
                        generator.doGenerate(state, fsa);
                        return Boolean.TRUE;
                    }
                });
                 
            }
        }
        return null;
    }
 
    @Override
    public boolean isEnabled() {
        return true;
    }
 
}
```
