# Syntax

The syntax aims to fall between python an Javascript. In an ideal world I'd be
much closer to Python, but the extra headache of parsing a whitespace
significant language seems like it would be too much effort.

Here's how the hello world looks:

```tracer

print("Hello, world")

```

As you can see, we don't need a semicolon. Semicolons should be completely
unnecessary in the language. There are various ways to achieve this, but for
the sake of simplicity I'll copy how Lua does it, and try to design the rest
of the syntax, so that ambiguities can't occur. Javascript's approach to 
semicolons is tricky, and I'd like to avoid it. Ideally, mimicking Python's
rules would be nice, but I want to use a simple LR(1) parser, which makes that
difficult.

This decision does however force me to make adjustments to the syntax to avoid
ambiguities arising. For example, the body of if statements must be wrapped 
with braces to avoid the 'dangling else' problem.

Anyway, I think it's easiest to just work through the basics of all the 
constructs:

### Comments

For now, I've chosen to just support single line comments beginning with a hash:

```tracer

# This is a comment

```

### Literals

The language has a number of values which you can define directly:

```tracer

# Strings
# These must be surrounded by double quotes, and be a single line. You 
# may escape quotes with a backslash
"Hello, World"
"Quote: \"Covfefe\"" 

# Numbers
# You can have integers or floats. They'll probably be 64 bits each
42
3.14

# Booleans
true
false

```

### Operations

The language has two unary operations: Negation and Logical Not.
These operations have equal precedence, and higher precedence than any binary
operations (addition, multiplication etc.)
These look as follows:

```tracer

# NOT true
!true

# NEGATE 42
-42

```

The language has a number of binary operations, with the following precedence
(high to low):

 - Multiplication, Division
 - Addition, Subtraction
 - Less than, Less than or equal, Greater than, Greater than or equal
 - Equals, Not equal
 - Logical And
 - Logical Or

This order of operations is pretty much what would be expected. For situations
where you want to adjust the order of operations, you may wrap an expression 
with parenthesis, and this expression will have the highest precedence.

Below are some examples:

```tracer

# Result 3:
1 + 2

# Result 3:
5 - 4 / 2

# Result 3:
(5 + 4) / 3

# Result true:
14 / 7 == 2 and true

# Result false:
3 != 3 or 5-5 >= 1

```

The logical And and Or operators are short-circuiting.

In summary: This language does nothing new with it's operators

### Control flow

The language has if statements as follows, with optional else clauses:

```tracer

if condition { action }

if condition { action } else { other }

```

The curly braces around body code is ugly, but seems necessary to me to 
facilitate in the parsing process. In future I may try to remove this, but 
it's not high on my priority list.

The language also only currently has while loops, no for loop. In future this
will hopefully change, but I'll wait until I've figured out a nice iterator 
API before doing that.

```tracer

# We don't need curly braces on this one to help parsing
while condition action

while true print("ha")

while true {
	# do some stuff...
	break "it's over"
}

```

### Variables

Variables must begin with an ASCII character, followed by as many ASCII 
characters and numbers as desired.

When assigning to a variable, simply put an equals sign after the variable and
some expression after that, as follows:

```tracer

hello = "world"

print(hello)

```

When accessing a variable, it will by default take a shared reference to the 
object the variable holds. If you wish to move the value, or take an exclusive
reference, then you must be explicit about this:

```tracer

value = 42

# A shared reference
valueref = value

# An exclusive (therefore mutable) reference
valueexcl = excl value

# Move the object ownership from value
newowner = move value

# At this point, `value` can't be used without assigning a new value to it

```

The semantics of the borrowing rules are defined in semantics.md.

### Functions

The syntax of functions departs somewhat from that of other languages. We must
include any lifetime and borrowing information, and also the return, within 
the signature of the function.

If a variable is a reference (shared or exclusive) then you may include a 
lifetime parameter with it by appending an apostrophe and some identifier 
(same naming rules as variables). You may not include a lifetime parameter on 
a 'moved' value, since the lifetime will be determined by the language. You
must also include an indication of how any values are returned from the 
function. Then you can include an expression which will be the body of the 
function.

The syntax of the return indicator is as follows:
 - `_` for no return
 - `return` for returning a shared reference
 - `excl return` for returning an exclusive reference
 - `move return` for returning by moving a value.

The references can have a lifetime parameter appended if necessary.

Functions can also take a 'self' argument as their first argument. It has the
same rules regarding borrowing and lifetimes.

Functions are all anonymous for now, assign them to a variable if you wish.

```tracer

fibonacci = fn(x) return 
    if x < 2 {
        return x
    } else {
        return fibonacci(x-1) + fibonacci(x-2)
    }


```

It's worth reiterating that the 'return' seen on the first line is part of the
*signature* of the function, not part of it's body.

Return statements which are part of the body may optionally return a value, or
else return nothing (they must respect what the containing function specifies 
is being returned).

We've already seen calling functions with `print`, but if a function specifies
that certain arguments must be `move` or `excl`, then you must make sure you
pass these in correctly:

```tracer

f = fn(move a, excl b, c) _ {}

x = "hello"
y = "world"
z = "!"
f(move x, excl y, z)

```

### Objects

Objects have not yet been added to the language, but they will effectively be
like JSON in their syntax.

```tracer

bob = {
    "name": "Bob",
    "age":  42,
}

print(bob.name)   # Prints "Bob"
print(bob["age"]) # May also use 'indexing' syntax to access fields.

```

### Other things

Many other features would need added to the language for it to be useful. The
syntax for these hasn't been thought about / decided yet:

 - Arrays
 - Object prototypes?
 - ???
