---
layout: page
title: Monto IR
permalink: /ir/
---

All intermediate representations are defined as JSON to reduce the effort of implementing custom parsers.

# Source Message

Source messages are used to transfer the contents of source documents. They consist of a unique name that identifies the document, an optional locial name like a qualified class name, a strictly increasing version number, the language and the contents of the document.

The contents of logical must not be set by the IDE, only by language services.

```
SourceMessage ::= {
  name: String,
  logical?: String,
  version: Int,
  language: String,
  content: String
}
```

## Example

```json
{
  "name": "myproject/nl/tudelft/hello/Hello.java",
  "logical": "nl.tudelft.hello.Hello",
  "version": 3,
  "language": "java",
  "content": "public class Hello {\n String world = ..."
}
```

# Product Message

```
ProductMessage ::= {
  
}
```