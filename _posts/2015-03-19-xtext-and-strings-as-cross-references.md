---
layout: post
title: Xtext and Strings as Cross-References
date: 2015-03-19 11:42:30.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Cross-Reference
- ProposalProvider
- STRING
- Xtext
meta:
  _edit_last: '25129546'
author: Christian Dietrich
---
Cross References are a often used concept in Xtext. They ususally work like this

```
Model:
    definitions+=Definition*
    usages+=Usage*
;
 
Definition:
    "define" name=ID
;
 
Usage:
   "use" definition=[Definition]
;
``` 

They can be used in the model like this
```
define Thing
use Thing
```
but what if i want to write something like
```
define "This is a Thing"
use "This is a Thing"
```
well the definition part is easily changed
```
Definition:
    "define" name=STRING
;
```
But what about the usage part?
well it is quite easy as well. refName=[Type] is short for refName=[Type|ID] which means ‘Refererence a Type and parse an ID. So to use another Terminal or Data Type Rule we change it to refName=[Type|RULENAME]
```
Usage:
   "use" definition=[Definition|STRING]
;
```

Now the cross refs are working fine. but if we try the editor we find out what autoedit and content assist disturb each other. We type " and auto edit gets us to "|". If we now type Crtl+Space for content assist we finally get "This is a Thing"" with an extra " at the end.
To avoid this we have to tweak the proposal provider a bit.
```
package org.xtext.example.mydsl.ui.contentassist
 
import org.xtext.example.mydsl.ui.contentassist.AbstractMyDslProposalProvider
import org.eclipse.emf.ecore.EObject
import org.eclipse.xtext.Assignment
import org.eclipse.xtext.ui.editor.contentassist.ContentAssistContext
import org.eclipse.xtext.ui.editor.contentassist.ICompletionProposalAcceptor
import org.eclipse.xtext.ui.editor.contentassist.ICompletionProposalAcceptor.Delegate
import org.eclipse.jface.text.contentassist.ICompletionProposal
import org.eclipse.xtext.ui.editor.contentassist.ConfigurableCompletionProposal
 
class MyDslProposalProvider extends AbstractMyDslProposalProvider {
 
    override completeUsage_Definition(EObject model, Assignment assignment, ContentAssistContext context, ICompletionProposalAcceptor acceptor) {
        super.completeUsage_Definition(model, assignment, context, new  StringProposalDelegate(acceptor, context))
    }
 
    static class StringProposalDelegate extends Delegate {
 
        ContentAssistContext ctx
 
        new(ICompletionProposalAcceptor delegate, ContentAssistContext ctx) {
            super(delegate)
            this.ctx = ctx
        }
 
        override accept(ICompletionProposal proposal) {
            if (proposal instanceof ConfigurableCompletionProposal) {
                val endPos = proposal.replacementOffset + proposal.replacementLength 
                if (ctx.document != null && ctx.document.length > endPos) {
                    // We are not at the end of the file
                    if ("\"" == ctx.document.get(endPos, 1)) {
                        proposal.replacementLength = proposal.replacementLength-1
                        proposal.replacementString = proposal.replacementString.substring(0,proposal.replacementString.length-1)
                    }
                }
            }
            super.accept(proposal)
        }
 
    }
 
}
```
what we basically do is detecting the situation and remove the extra `"`
