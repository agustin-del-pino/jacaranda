# 🌺 Jacaranda 🌺

Write jacaranda-lang then transpile to any code!


# What's Jacaranda?

It's a pseudo-language that pretends to be transpiled to some other language or some other thing (a diagram, perhaps).

## Little Example
Pet interface in Jacaranda Lang.

```
interface Pet: 
    prop string Name
    prop int Age
    method void Eat (float amountOfFood)
    method bool IsHungry
```

### Lexer

First, the Jacaranda-Lexer gets into action. Reads each char and creates the tokens. *See the Token Type Table further bellow*.
The final product will be some like this:

```
(
    [KYW-INTERFACE]
    [IDT Pet][DLT-SEMICOLON]
    [KYW-PROP][KYW-TYPE-STRING][IDT Name]
    [KYW-PROP][KYW-TYPE-INTEGER][IDT Age]
    [KYW-METHOD][KYW-TYPE-VOID][IDT Eat]
        [DLT-PAREN-LEFT][KYW-TYPE-FLOAT][IDT amountOfFood]
    [KYW-METHOD][KYW-TYPE-BOOL][IDT IsHungry]
)
```

### Parser
Once the tokens were created, the Jacaranda-Parser does its part. *The syntax rules are explained further bellow*.

The abstract result of the parse is some like this:
```
        Independent-Structure-Node
           /            \
       Id-Node        Content-Node
        /  \               |
    Type    Name    *Dependent-Structure-Node
```

There are two kind of **Structure-Node**s: Independent and Dependent. The logic is simple, the second ones depends on the first one. So, at root level, can't exists a Dependent Structure.

Then, the **User Implementation** is use it. As user, you will have the AST for the transpilation.

### Transpilation
Let's use the case of TS Transpilation.

```py
import jacaranda

ts = jacaranda.Transpiler()

@ts.transpile(jacaranda.PROP)
def ts_prop(p: jacaranda.PropNode):
    return f"{p.id.name}: {p.id.type}"

@ts.transpile(jacaranda.PARAMS)
def ts_param(ps: jacaranda.ParamsNode)
    return ",".join([f"{p.id.name}: {p.id.type}"])

@ts.transpile(jacaranda.METHOD)
def ts_method(m: jacaranda.MethodNode):
    params = ts(m.params.nodes)
    content = ts(m.content.nodes)
    return f"{m.id.name}" + f"({params})" if params else "" + " -> {m.id.type}" + f"{{\n{content}\n}}" if content else ""
 
@ts.transpile(jacaranda.INTERFACE)
def ts_interface(i: jacaranda.InterfaceNode):
    declaration = ts(i.content.nodes)
    return f"interface {i.id.name} {{\n{declaration}\n}}"

if __name__ == '__main__':
    list_of_nodes = jacaranda.lex_and_parse("""
    interface Calculator:
        prop number[] history
        method number sum(number a, number b)
        method number sub(number a, number b) 
    """)

    typescript_code = ts(list_of_nodes)
    
    print(typescript_code)
```

# Jacaranda Lang Syntax

This lang is strongly based on keyword and tabbing syntax. So every "statement" it will start by a keyword that represent its kind. Each keyword has its own syntax rules but in general, it can be subtracted the following rule:

```
kind identifier : meaning
```

### Kind
It's the keyword, i.e: `interface`, `class`, `prop`

### Identifier
It's the sentence right after the keyword, in general, this part is compounded by two sub-parts: *Type* and *Name*.

### Semicolon
It's the separation between the declaration and the assignation of the meaning.

### Meaning
It's the 'content' of the sentence, its body. What kind of sub-statements can be contained in the 'statement' it's determinate by its kind.

# Keywords

There're two big groups of keywords: compounded and component.
The first ones are keywords that are compounded by sub-statements. The second only can exists inside of the firts ones.

## Compounded Keywords.

### Object

*It can contain properties and methods*.

```
object [name]:
    ...
```

### Interface

*It can contain properties, methods and generic types*.

```
interface [name]<generics>:
    ...
```

### Class

*It can contain properties, methods and generic types*.

```
class [name]<generics>:
    ...
```

### Abstract Class

*It can contain properties, methods and generic types*.

```
abstract [name]<generics>:
    ...
```

### Function

*Only can contain its return type*.

```
func [name](params): {return-type}
```

## Component Keywords.

### Property

*Only can be placed at: Objects, Interfaces, Classes, Abstract Classes*.

```
prop {type} [name]
```

### Method

*Only can be placed at: Objects, Interfaces, Classes, Abstract Classes*.

```
method {return-type} [name](params)
```

# Relations

The statements can be related between them.

There are two kind of relation: *by communication*
and *by composition*. The first one refers to when the statements share information. The second one can be resumed as inherency.

## By Communication

Let a function `calc` that depending on one of its params it will execute addition or subtraction operation for two numbers that also are passed by its params.

```
func calc(number a, number b, string op): number 
```

The operations before mention it are two separated function as well.

```
func add(number a, number b): number
func sub(number a, number b): number
```

So, when *op* is '+' the `calc` function have to call the `add` function, otherwise have to call the `sub` function.

For doing that it have to use it the `when` keyword and `otherwise` keyword. For call to a function just do that, use the `call` relation.

```
func calc(number a, number b, string op):
    when op is '+':
        call add(a, b)
    otherwise:
        call sub(a, b)
```

But now the return type was lost, so use the `return` keyword before the `call`. By this way, the `calc` function it will return the same type of the called function.

*In case the called functions haven't the same return type, the principal function it will return the two types*.

```
func calc(number a, number b, string op):
    when ops is `+`:
        return: call add(a, b)
    otherwise:
        return: sub(a, b)
```

## By Composition

This is more natural for people that are used to OOP. This kind of relation is the classic `extends` and `implements`.

Let an interface `Figure`.

```
interface Figure:
    prop number hight
    prop number width
    method number area
```

Now, let's implements this interface by using the `implements` relation.

```
implements Figure: Rectangle
```

In a more classic way, it must be created a class called rectangle and specify its implementation. But is some kind of redundant because the implementations of any interface it's always a class. So, just indicate which are the implementations.

However, it can be declared a class inside an `implements` relation. This use-case is when the class has non-public method or properties that must be specified.

```
implements Figure:
    class Square:
        section private:
            method void fixedSize() 
```
In this case it was used the `section` keyword. This allow to encapsulate many statements. This section has `private` as name. 

Analogously as `implements` it can be use the `extends` relation.

```
class Rectangle:
    prop number hight
    prop number width
    method number area

extends Rectangle:
    class Square:
        same hight and width: setter
```

Also, for this example, it's using the `same` relation, this allow to relate properties indicating what kind of relation have between them. In this case is `setter`. So, the value of hight and the value of width it will be the same for both properties.

# Documentation
One of the principles of sustainable code is the documentation that explains its behavior.

There're three kind of documentation keywords: Comment, Doc, Spec.

## Comment
As usual, this kind of keyword is mere descriptive for humans. It will be ignored when at Lexer phase.
```
object CoolSingleton {
    comment TODO: "Do something cool."
}
```

## Doc
This keyword works for document parts of the code, for example.

```
test Fetch:
    doc Given: "A url."
    doc When: "GETs the json."
    doc Then: "200 OK and the json OK."
```

## Spec
This keyword is used for specify the code. 

```
spec Description: "API for manage a Server"
interface ServerAPI:
    spec Method:
        param port: "the port of the server"
        returns: "the operation code"
    method number Start(int port)
```

# Even further beyond

Jacaranda-lang is highly configurable and extensible, so all keywords can be replaced for another. And of course, can add new keywords. *The original behavior of the default keywords can't be changed*.

## Jacaranda Configuration
The configuration is given to the transpiler by a `.yml` config file. The schema is something like this:

```yml
jacaranda:
    keywords:
        Object: "object"
        Interface: "interface"
```

Let's change some few things. The keyword for the Object it will change to `obj`, also let's add a new keyword: `const`.

```yml
jacaranda:
    keywords:
        Object: "obj"
        Interface: "interface"
        Constant: `const`
```

With this, the Lexer will recognize those words as keywords but need to add the Parser part for understand what is the behavior of the new `Constant` statement.

```yml
jacaranda:
    keywords:
        Object: "obj"
        Interface: "interface"
        Constant: `const`
    behavior:
        Constant:
            type: "Single"
            node-type: "ConstantNode"
            identifier:
                - "type"
                - "name"
            meaning:
                allowed-tokens:
                    - LIT-INT
                    - LIT-FLT
                    - LIT-NUM
                    - LIT-STR
```

By this configuration, Jacaranda will accept the `Constant` statement by the `const` keyword. The type of the statement is "single" and its node type is "ConstantNode". The identifier node needs two tokens: the type and the name. In the meaning section just indicates what kind of tokens are allowed to be part of it.

