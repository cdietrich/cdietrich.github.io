---
layout: post
title: Xtext Content Assist Auto Activation
date: 2011-09-19 20:13:14.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Xtext
- Auto Activation
- Content Assist
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---
Xtext offers nice Content Assist facilities. JDT offers a nice additional feature: Content assist is autoactivated if a certain character (.) is typed. To activate this feature in Xtext simply customize your UiModule
```
public class MyDslUiModule extends org.xtext.example.mydsl.ui.AbstractMyDslUiModule {
    public MyDslUiModule(AbstractUIPlugin plugin) {
        super(plugin);
    }
     
    @Override
    public void configure(Binder binder) {
        super.configure(binder);
    	binder.bind(String.class)
			.annotatedWith(com.google.inject.name.Names.named(
			(XtextContentAssistProcessor.COMPLETION_AUTO_ACTIVATION_CHARS)))
			.toInstance(".,:");
    }
}
```
In this case content assist is autoactivated on `.` `,` and `:`