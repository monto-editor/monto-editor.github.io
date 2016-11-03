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

Each source message contains a `name` that uniquely identifies the source-code artifact, a unique ID `version` that identifies the current version of the source artifact, the language used in the source artifact `language`, and the content of the source artifact `content`. The Monto plugin is responsible for using a unique `name` and a fresh `version` number in every source message it sends. Optionally, `logical_name` can be set and refered to by services. For example, in Java the `logical_name` field is set to the qualified class name.

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

Services send their result encoded in JSON as product messages back to the broker. Product messages have the following format:

```
ProductMessage ::= {
  name: String,
  logical_name?: String,
  version: Int,
  product: String,
  content: JSON
}
```

The `name` and `version` field of a product message link it to a specific version of a source-code artifact the product was derived from. The field `product` describes the sort of product that is contained in this message and the `content` field contains the actual pay-load of the service.

### Example

```json
{
  "name": "myproject/nl/tudelft/hello/Hello.java",
  "logical_name": "nl.tudelft.hello.Hello",
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

## Syntax Highlighting

Syntax highlighting selects different fonts for different parts of a source file. Typically, syntax highlighting associates a font (family, color, weight, etc.) with specific syntactic or semantic classes of code elements featured in the edited language. The IDE then displays each code fragments in the selected font.

For Monto IR, we must abstract all language-specific and IDE-specific aspects of syntax highlighting. In particular, Monto IR cannot refer to language-specific classes of code elements. Instead, we opt for a low-level description of syntax highlighting using messages of the following format:

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

Syntax highlighting is described as a sequence of non-overlapping highlighted tokens. Each token is identified by its character offset in the source document and the number of characters it includes. The font used for highlighting through attributes identical to those in CSS.

Not all language services require the full flexibility of the highlighting IR. However, services are free to confine themselves to use only a subset of the font attributes. Conversely, not all IDEs support the full flexibility of our highlighting IR. For example, many IDEs do not support multiple font families within a single text editor. However, IDEs are free to ignore those parts of the IR that they cannot render. Finally, the coloring of syntax sometimes is IDE-specific, for example, because the IDE uses a dark theme for its GUI components.

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

<img src="../images/highlighting.png" width="300"/>

## Abstract Syntax Tree (AST)

An AST represents the syntactic structure of a program. While an AST is not directly useful for visualization in an IDE, it serves as an intermediate result that can be shared by services: A parser service can produce the AST and other services can require the AST as input. To support this scenario, we designed the following Monto IR for representing ASTs:

```
AST ::= {
  name: String,
  offset: Int,
  length: Int,
  children: AST*
}
```

The AST representation mostly corresponds to s-expressions: AST nodes feature a name and a list of children. In addition, AST nodes provide a pointer into the source document, encoded through offset and length. This way, other services can extract the original text from the source document when needed.

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

To support arbitrary languages, the Monto IR cannot make any assumption on the structure of the outline. For example, the Monto IR must support languages that mostly consist of top-level declarations like Haskell as well as languages that use more deeply nested structures like Java. To this end, we designed the following IR for outline views:

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

The root of the outline IR is a list of outline items, such that multiple top-level declarations can be displayed. Each item defines a text label for display in the outline. The label is free-form and can contain typing information in addition to the name of the declaration. We declare the icon of an outline item through an optional HTTP URL. This way, an IDE can load and cache the icons for outline items. To represent nested structures, an outline item can also declare a list of subitems. Finally, each outline item provides a pointer into the source document, encoded through offset and length. For our example Java class, the Java outline service sends the following message:

```json
[
  {
    "label": "Hello",
    "icon": "http://localhost:8080/class-public.png",
    "offset": 13,
    "length": 5,
    "children": [
      {
        "label": "world : String",
        "icon": "http://localhost:8080/field-default.png",
        "offset": 30,
        "length": 5
      },
      {
        "label": "main(args) : void",
        "icon": "http://localhost:8080/method-public.png",
        "offset": 68,
        "length": 4
      }
    ]
  }
]

```

#### Result:

<img src="../images/outline.png" width="300"/>

## Error Reporting


Many language services detect and report errors, warnings, or other information about a program. Example services that produce such messages include parsers, spell checkers, type checkers, static analyses, and unit testing. Besides providing a list of such messages, IDEs typically highlight the location of the messages directly in the source code to provide visual feedback to the user. Moreover, IDEs typically display a detailed description of the message when the user hovers with the mouse over a highlighted location in the code. Error reports can be represented with the following IR:

```
Report ::= Message*
Message ::= {
  offset: Int,
  length: Int,
  level: Level,
  category: String,
  description: String
}
Level ::= "info" | "warning" | "error"
```

A report consists of a sequence of messages. Each message links to a region in the source code via offset and length and each message declares a severity level, which is either `info`, `warning`, or `error`. Finally, each message declares a message category (such as spelling, type, test, etc.) and each message contains a detailed description. The category enables users to filter for messages in the global message list within the IDE. A service that applies a lint-style analysis could send the following report, where the first message marks an error and the second one marks a warning:

```json
[
  {
    "offset": 0,
    "length": 0,
    "level": "error",
    "category": "lint",
    "description": "Class 'Hello' must not be in default package."
  },
  {
    "offset": 23,
    "length": 12,
    "level": "warning",
    "category": "lint",
    "description": "Field 'world' is public but non-final."
  }
]
```
