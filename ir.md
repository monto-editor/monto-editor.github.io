---
layout: page
title: Monto IR
permalink: /ir/
---

All intermediate representations are defined as JSON to reduce the effort of implementing custom parsers.

## Source Message

An IDE sends a source message to the language services upon a change to the source code. The format of the source message is fairly straightforward:

```
SourceMessage ::= {
  name: String,
  logical_name?: String,
  version: Int,
  language: String,
  content: String
}
```

Each source message contains a `name` that uniquely identifies the source-code artifact, a unique ID `version` that identifies the current version of the source artifact, the language used in the source artifact `language`, and the content of the source artifact `content`. The Monto plugin is responsible for using a unique `name` and a fresh `version` number in every source message it sends. The `logical_name` might be set by services to for example the qualified class name.

### Example

```json
{
  "name": "myproject/nl/tudelft/hello/Hello.java",
  "logical_name": "nl.tudelft.hello.Hello",
  "version": 3,
  "language": "java",
  "content": "public class Hello {\n String world = ..."
}
```

## Product Message

Services send their results encoded in JSON as product messages to the broker. The `source` and `version` field referr to the source artifact they were derived from. The `product` field describes the product contained in the message.

```
ProductMessage ::= {
  source: {
    physical: String,
    logical?: String
  },
  version: Int,
  product: String,
  content: JSON
}
```

### Example

```json
{
  "source": {
    "physical": "myproject/nl/tudelft/hello/Hello.java",
    "logical": "nl.tudelft.hello.Hello"
  },
  "version": 3,
  "language": "java",
  "product": "highlighting",
  "content": [
    { "offset": 0, "length": 12,
      "font": { "color": { "red": 3, "green": 101, "blue": 192 } }
    }
  ]
}
```

## Syntax Highlighting IR

The Monto IR for syntax highlighting is a sequence of non-overlapping highlighted tokens, each identified by a character offset and length. The attributes for highlighting are inspired by CSS.

```
Highlighting ::= Token*

Token ::= { offset: Int, length: Int, font: Font }

Font ::= {
  color?: Color, bgcolor?: Color,
  family?: String, size?: Int,
  style?: String,  variant?: String,
  weight?: String
}

Color ::= { red: Int, green: Int, blue: Int }
```

### Example

```json
[
  {
    "offset": 0, "length": 12,
    "font": {
      "style": "bold",
      "color": { "red": 3, "green": 101, "blue": 192 }
    }
  },
  {
    "offset": 13, "length": 24,
    "font": {
      "color": { "red": 0, "green": 0, "blue": 0 }
    }
  },
  {
    "offset": 38, "length": 7,
    "font": {
      "style": "italic",
      "color": { "red": 0, "green": 136, "blue": 43 }
    }
  },
  ...
]
```

#### Result:

<img src="../images/highlighting.png" alt="Monto, a solution to the iDE portability problem" width="300"/>

## Abstract Syntax Tree IR

While an AST is not directly useful for visualization in an IDE, it serves as an intermediate result that can be shared by services: A parser service can produce the AST and other services can require the AST as input.

```
AST ::= {
  name: String,
  offset: Int,
  length: Int,
  children: AST*
}
```

AST representation mostly corresponds to s-expressions: AST nodes feature a name and a list of children. In addition, AST nodes provide a pointer into the source document, encoded through offset and length. This way, other services can extract the original text from the source document when needed.

### Example

```json
{
  "name": "CompilationUnit", "offset": 0, "length": 150,
  "children": [
    {
      "name": "ClassDeclaration", "offset": 0, "length": 149,
      "children": [
        { "name": "Modifiers", "offset": 0, "length": 6 },
        { "name": "Identifier", "offset": 13, "length": 5 },
        { "name": "FieldDeclaration", "offset": 23, "length": 23 },
        { "name": "MethodDeclaration", "offset": 49, "length": 98 }
      ]
    }
  ]
}
```

## Outline Views

An outline view provides a structural overview of a source document, typically in a hierarchical form. For example, an outline for Java shows type declarations, fields, and methods. IDEs often display a representative icon in front of each outline item to illustrate the kind of the item. Moreover, when the user clicks on an outline item, IDEs often move the cursor to the definition site of the item in the source document.

```
Outline ::= Item*

Item ::= {
  label: String,
  icon?: URL,
  children: Item*,
  offset: Int,
  length: Int
}
```

