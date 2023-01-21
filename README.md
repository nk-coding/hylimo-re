# HyLiMo Requirements Elicitation Information & Overview

> **TL;DR**: HyLiMo is a Tool for hybrid (both textual and graphical) modeling of UML class diagrams. During the meeting, we discuss what the DSL for class diagrams could look like, and which features can be supported in the graphical view.

## Background

Creating UML class diagrams is a common task for software engineers.
Therefore, several tools already exist for this purpose, e.g. MermaidJS or diagrams.net.
Most of these tools can be put into two categories:
1. **Descriptive DSL, Autolayout**: A External DSL is used to describe the structure of the Diagram, the graphical representation is then generated based on the given DSL text. Typically, manual layouting is not at all or is very limited supported.
1. **Graphical editor**: The diagram is created using a graphical editor, typically with completely manual layouting (or limited autolayout support)

As both approaches have their benefits, recently, tools using hybrid/blended modeling appeared, e.g. [D2](https://d2lang.com/tour/intro/) and [Structurizr](https://structurizr.com/). These tools typically use a descriptive external DSL, and allow to add/remove and update the contents of diagram elements (e.g. classes) in the graphical view.

However, manual & precise layouting is **not supported** as only autolayouting is implemented, forcing users to use less efficient approaches when precise or custom layouting is required.

## HyLiMo

HyLiMo is a tool for creating diagrams using a hybrid editor consisting of a 
- embedded/internal DSL to describe the diagram
- graphical view to manipulate parts of the diagram, most important layouting

Technically, it works like this:
- The diagram is described using an embedded/internal DSL in the general-purpose scripting language Syncscript.
- The DSL text is executed to generate the diagram model.
- The model is displayed in the graphical view.
- When the user manipulates the graphical view, e.g. by moving a class, the DSL code is updated to correspond to the graphical changes.
- Then, the User can do more textual or graphical changes.

To see what this flow looks like, have a look at the following video:

https://user-images.githubusercontent.com/53957498/213848406-a562ac8e-3847-41b5-8abc-81794f6ae17d.mp4

## Features

This approach results in some unique features
- The DSL text acts as a single source of truth: diagrams can be shared as version-controlled documents
- Manual layouting is both supported and easy thanks to the graphical view
- Precise layouting can be done by manipulating layout-related information in the DSL code
- HyLiMo uses an embedded DSL: therefore, functions and control flow statements can be used to simplify creating the diagram
- The diagram is fully stylable, using a simple CSS-like syntax
- HyLiMo is based only on web technologies and can run in both browser and NodeJS environments making integration into IDEs and command line tools easy

## Requirements Elicitation Interview

By now, the underlaying technology stack is implemented.
I will use the RE interviews to collect features and (design) ideas regarding
- the UML class diagram DSL
- the graphical editor

In a one-on-one format, I will explain the general concept again, and give a demonstration of the already working parts.
Then, we will discuss how the DSL could look like, based on some existing ideas (see below), which graphical editing features should be supported, and how those interactive features can be implemented.

Based on the protocols of those meetings and my own ideas, I will then create a questionnaire to
- evaluate the importance of specific features
- find the most popular approach where multiple exist

## DSL Example

```
classdiagram {
    House = class("House") {
        "roomCount" : Int
        "wallColor" : "Color"
        "createRoom" : fun("floor" : Int) => "Room"
    } at pos(0, 100)

    Room = class("Room") {
        +("size") : Double
    }

    House --> Room {
        start = 0.5
        over = list(pos(200, 300), pos(400, 500))
        end = 0.5
    }

    styles {
        cls("class") {
            minWidth = 100
            maxWidth = 600
        }
    }
}
```

## Existing questions

Following are some of the already existing questions I try to answer during RE.
Feel free to already think about those before the meeting.
Of course, you can also bring and/or come up with your ideas for these or related questions.

- Rendering
    - how to render visibility modifiers of fields and methods
    - which types of lines should be supported for relations (bezier, direct, ...)
- DSL
    - how to define class fields (following some possible examples)
        - `+("size") : Int`
        - `+("size" : Int)`
        - `field(+, "size", Int)`
    - how to define class methods (following some possible examples)
        - `+("createRoom") : fun("floor" : Int) => "Room"`
        - `+("createRoom") : fun("floor" : Int) : "Room"`
        - `+("createRoom" : fun("floor" : Int) => "Room")`
        - `fun(+, "createRoom", list("floor" : Int), "Room"`
        - ```
          +("createRoom") {
            "floor" : Int
          } : "Room"
          ```
        - ...and many more combinations of these ideas
    - support multiple separated (by line) areas (of fields and methods) in a class?
    - which visibility modifiers should be supported
    - how to declare a relation between classes (or in general line segments)
        - segments can be of different types, e.g. bezier, direct line, ...
        - some helpers could make defining those simpler, e.g. something like `polyline(p1, p2, p3)`
    - how to support stereotypes for classes (and which?)
- Graphical view
    - how & when to display the points which can be manipulated by moving around
    - which features, apart from moving (classes and other points) and resizing (classes) should be supported (e.g. changing text values)
    - should control features (e.g. resize helpers, movable points) scale with the zoom level of the diagram or not?
- SyncScript
    - (context: in SyncScript, everything is an expression) should look control structures evaluate to
        - null
        - the last inner value
        - a list of all inner values

## What does already work?

You can find the hosted version of HyLiMo here: https://hylimo.github.io

You can put there [this example diagram](TODO).

:warning: Caution: Currently, only my underlying graphics framework and parts of the graphical editor are implemented. You can see here how one can use this framework to create diagram elements. Later, the user will use the DSL designed during these RE meetings to create the diagram, this DSL will then use the underlying graphics framework you see here (and not the modeling user!).

## SyncScript

SyncScript is the general-purpose programming language in which the class diagram DSL is embedded.
It is a rather simple, interpreted, dynamically typed scripting language which focuses on
- syntactic flexibility for creating internal/embedded DSLs: Scala-like higher-order functions, custom operators
- tracing: to update the DSL code based on graphical updates, one needs to know which Expression in the code caused which value
- web support: everything, from lexer to interpreter is implemented using web technologies only

Following is a very short introduction to SyncScript, in case you are interested.

### Hello world
```
println("Hello world")
```

### Data types
- strings
- numbers
- objects
    - an object can have fields indexed by both integers and strings
    - objects can have prototypes. If a field is not found on the object itself, the interpreter looks at the prototype and tries to find it there. Fields can be accessed using the common `.` syntax.
- functions (see below for more details)
- null
- boolean
    - technically, booleans are implemented as objects

### Variables

Variables are assigned using the `=` operator.

Example:
```
i = 1
test = "Hello world"
```

### Functions

Functions are the most important concept in SyncScript, as all control-flow structures are implemented using functions.

To declare a function, use curly brackets:
```
printHelloWorld = {
    println("Hello world")
}
```

Functions can be invoked using round brackets:
```
printHelloWorld()
```

Arguments are provided using the `args` identifier, SyncScript supports both index-based and named arguments:
```

printSth = {
    println(args.text)
}
printSth(text = "Some text")
```

To simplify accessing indexed-based arguments, destructuring can be used:
```
createPoint = {
    (x, y) = args
    "TODO"
}
createPoint(10, 20)
```

Additionally, the first positional argument can be accessed under the name `it`:
```
myPrintln = {
    println(it)
}
myPrintln("test")
```

As a return value, the value of the last expression in the function body is used.

SyncScript uses static scoping, each function creates a new scope. You can use `this` to access the current scope directly

### Higher-order functions

To allow the creation of the DSL constructs shown above, higher-order functions can be used. Similar to Scala and Kotlin,
```
point {
    x = 1
    y = 2
}
```
is identical to
```
point({
    x = 1
    y = 2
})
```

It is even possible to use multiple of these blocks, as shown in the implementation of the builtin-function `if`:
```
if (aVariable) {
    println("the variable is true")
} {
    println("the variable is not true")
}
```

### Operators

Operators are functions, which get two positional arguments: the left and right side:
```
div = {
    (left, right) = args
    left / right
}
10 div 20
```

### Important built-in functions

- **if**
  ```
  if(condition) {
    println("true")
  } {
    println("false")
  }
  ```
- **while**
  ```
  while { a < b} {
    println("body")
  }
  ```
- **obj**
  ```
  aPoint = obj(x = 10, y = 20)
  ```
- **list**
  ```
  someNumbers = list(1, 2, 3, 4, 5)

  someNumbers.forEach {
    println(it)
  }
  someNumbers.add(100)
  lastElement = someNumbers.remove()
  ```
