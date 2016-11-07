---
layout: page
title: About
permalink: /about/
---

<img src="../images/monto-all.png" alt="Monto, a solution to the iDE portability problem" width="400"/>

Modern IDEs support multiple programming languages via plug-ins, but developing a high-quality language plug-in is a huge development effort and individual plug-ins are not reusable in other IDEs. This problem is called the _IDE portability problem_.

Monto provides a solution to the IDE portability problem based on a language-independent and IDE-independent intermediate representation ([IR](../ir)) for editor-service products. This IR enables IDE-independent language services to provide editor services for arbitrary IDEs, using language-independent IDE plug-ins.

Monto combines the IR with a [service-oriented architecture](../architecture) to facilitate the modular addition of language services, the decomposition of language services into smaller interdependent services, and the use of arbitrary implementation languages for services.

If you have questions, join us in our Gitter channel:
<a href="https://gitter.im/monto-editor/Lobby" rel="some text">![gitter](https://img.shields.io/gitter/room/nwjs/nw.js.svg)</a>
