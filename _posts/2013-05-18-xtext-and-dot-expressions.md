---
layout: post
title: Xtext and Dot/Path-Expressions
date: 2013-05-18 16:21:27.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Dot-Expresion
- Scoping
- Xtext
- Path
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---

If you have a DSL that describes structure (e.g. like an Entity DSL) you often need to “walk” on this structure using dot/path-expressions.

Let us asume we have a grammar like

```
Model:
    entities+=Entity*
;
 
Entity:
    "entity" name=ID "{"
        features+=Feature*
    "}"
;
 
Feature:
    Attribute | Reference
;
 
Attribute:
    "attr" name=ID ":" type=DataType
;
 
enum DataType:
    string | int
;
 
Reference:
    "ref" name=ID ":" type=[Entity]
;


```
and a model like
```
entity A {
    attr a1 : int
    attr a2 : string
    ref b : B
    ref c : C
}
 
entity B {
    attr b1 : string
    attr b2 : string
    ref a : A
    ref c : C
}
 
entity C {
    attr c1 : string
    attr c2 : int
}
```
and want to have expressions like
```
use A.b.b2
use A.b.c.c1
use A.a1
use A.b.a.a1
```
but how to do this with Xtext?
there are several possibility but the following was working well in my usecase:
```
Model:
    entities+=Entity*
    usages+=Usage*
;
Usage:
    "use" ref=DotExpression
;
 
DotExpression returns Ref:
    EntityRef ({DotExpression.ref=current}  "." tail=[Feature])*
;
 
EntityRef returns Ref:
    {EntityRef} entity=[Entity]
; 
```
a bit of scoping
```
class MyDslScopeProvider extends AbstractDeclarativeScopeProvider {
 
    def IScope scope_DotExpression_tail(DotExpression exp, EReference ref) {
        val head = exp.ref;
        switch (head) {
            EntityRef : Scopes::scopeFor(head.entity.features)
            DotExpression : {
                val tail = head.tail
                switch (tail) {
                    Attribute : IScope::NULLSCOPE
                    Reference : Scopes::scopeFor(tail.type.features)
                    default: IScope::NULLSCOPE
                }
            }
             
            default: IScope::NULLSCOPE
        }
    }
 
}
```
and it works fine.

as an additional note: the ast of the expressions look like

![AST]({{ "/assets/img/blog/new.png" | absolute_url }})
