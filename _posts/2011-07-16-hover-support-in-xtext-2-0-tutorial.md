---
layout: post
title: 'Hover support in Xtext 2.0: Tutorial'
date: 2011-07-16 18:54:49.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Xtext
- Hover
author: Christian Dietrich
---
Xtext 2.0 comes with an all new Hover API (see Christoph's Blog [http://ckulla.wordpress.com/2011/02/06/hover-support-in-xtext-2-0/](http://ckulla.wordpress.com/2011/02/06/hover-support-in-xtext-2-0/)). I want to give a short introduction on how to use what with Xtext's Greeting Example.

So first we create a new Xtext project with the wizard and generate the language. We start a runtime application and create a project with a model file. Here is what the default hover looks like:

![Default Hover]({{ "/assets/img/blog/default-hover.png" | absolute_url }})

We want is to adopt the hover support to look like this:

![Custom Hover]({{ "/assets/img/blog/default-custom.png" | absolute_url }})

We have to basically implement 2 interfaces: `IEObjectHoverProvider` to customize the header line and `IEObjectDocumentationProvider` to customize the content section. Here 2 simple implementations (using appropriate superclasses)

```
package org.xtext.example.mydsl.ui;
 
import org.eclipse.emf.ecore.EObject;
import org.eclipse.xtext.ui.editor.hover.html.DefaultEObjectHoverProvider;
import org.xtext.example.mydsl.myDsl.Greeting;
 
public class MyDslEObjectHoverProvider extends DefaultEObjectHoverProvider {
 
    @Override
    protected String getFirstLine(EObject o) {
        if (o instanceof Greeting) {
            return "Damn good greeting: " + ((Greeting)o).getName();
        }
        return super.getFirstLine(o);
    }
 
}
``` 

```
package org.xtext.example.mydsl.ui;
 
import org.eclipse.emf.ecore.EObject;
import org.eclipse.xtext.documentation.IEObjectDocumentationProvider;
import org.xtext.example.mydsl.myDsl.Greeting;
 
public class MyDslEObjectDocumentationProvider implements IEObjectDocumentationProvider {
 
    @Override
    public String getDocumentation(EObject o) {
        if (o instanceof Greeting) {
            return "This is a nice Greeting with nice <b>markup</b> in the <i>documentation</i>";
        }
        return null;
    }
 
}
```

Finally we have to bind these classes in the UiModule of our dsl. 

```
package org.xtext.example.mydsl.ui;
 
import org.eclipse.ui.plugin.AbstractUIPlugin;
import org.eclipse.xtext.documentation.IEObjectDocumentationProvider;
import org.eclipse.xtext.ui.editor.hover.IEObjectHoverProvider;
 
/**
 * Use this class to register components to be used within the IDE.
 */
public class MyDslUiModule extends org.xtext.example.mydsl.ui.AbstractMyDslUiModule {
    public MyDslUiModule(AbstractUIPlugin plugin) {
        super(plugin);
    }
 
    public Class<? extends IEObjectHoverProvider> bindIEObjectHoverProvider() {
        return MyDslEObjectHoverProvider.class;
    }
 
    public Class<? extends IEObjectDocumentationProvider> bindIEObjectDocumentationProviderr() {
        return MyDslEObjectDocumentationProvider.class;
    }
}
```

That is all we have to do to get customized hovers with Xtext 2.0. 