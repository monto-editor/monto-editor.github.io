---
layout: page
title: Architecture
permalink: /architecture/
---

Monto facilitates service-oriented architecture for connecting language services and IDEs, where all messaging is done via the network and all messages adhere to a simple protocol. This architecture enables

1. the modular addition of language services for new languages,
2. the decomposition of a language service into multiple smaller services that can use each other's results, and
3. the implementation of services and IDEs in any implementation language.

The central component of our architecture is the *message broker* that implements a service registry and routes messages between IDEs and language services.

### Example workflow

The figure illustrates the workflow of Monto through an example featuring two Java services. The parser service takes source code as input and produces ASTs. The outline service takes source code and an AST as input and produces an outline of the code.

<img src="../images/architecture.png" width="700"/>

When a user changes a source file (1), the Monto plug-in within the IDE sends a source message (2) to the message broker. Each source message contains a unique ID called version, the file name, the language, and source code of the source file. Editor products include the version ID of the source, such that we can easily discard outdated products.

The message broker is responsible for distributing source and product messages to services that take them as input. The broker distributes messages to services in parallel but respects dependencies between services. In our example, the broker first forwards the source message to the parse service (3). The Java parser extracts the source code from the source message and parses the code into an AST. The parser then encodes the AST as a product message according to the Monto IR and sends the message back to the broker (4).

Now the broker has all information required to trigger the outline service and forwards the source and AST message to the outline service (5). The outline service computes an outline, encodes it according to the Monto IR, and sends it back to the broker.

The broker forwards all product messages to the IDE as soon as they arrive (6). Inside the IDE, the Monto plugin interprets the Monto IR of the received product messages and updates the editor view of the IDE and the user observes the result (7). This workflow is repeated on every change to the source code.