# The Tracer Programming Language

This repository contains a rough working spec for how the language will work.

This will include syntax, semantics, and discussion on implementation.

The implementation is at https://github.com/jawm/tracer

## Overview

Tracer is a dynamically typed language. All values are objects. The lifetime 
of all objects is tracked, so as to be able to manage the memory used to store
them.

This model of memory management borrows heavily from Rust. Some adaptation is
necessary since the language is dynamic, which means much of the checks must
happen at runtime, but I am convinced that it should be viable.

This does force some extra concessions that most dynamic langauges wouldn't 
have. For example, I've got to introduce an ownership model for all objects.
Any object must have one owner. The object can also be borrowed, either with a
shared borrow (hence read-only), or an exclusive (allows mutation). The 
duration of all borrows will be tracked too, and given lifetimes. During these
lifetimes, the owner of the object won't be allowed to use it. These rules
allow us to prevent data races from occuring. This should allow for higher
performance concurrent code, unlike in Python with it's GIL.

Syntactically the language aims to fall somewhere between Python and 
Javascript, although this is somewhere that I'm willing to make some 
concessions for the sake of advancing other parts of the language.

Some other features of the language is that I will endeavour to avoid using
exceptions wherever possible, instead returning a `Result` type similar to 
Rust. Likewise there won't be a `null` type, instead we will use an `Option`
object to wrap nullable values.

In some cases, exceptions will be necessary. I will try to restrict these to
situations where the issue is that the programmer has written bad code (e.g
passing the wrong number of arguments to a function). In the initial version
of the language these exceptions will actually just cause the langauge to 
panic. I think this is justified, since essentially they should only occur for
avoidable errors. Further down the line, I may add proper exception handling
with some sort of try/catch mechanism.

I'll try to give an overview of the syntax in syntax.md
