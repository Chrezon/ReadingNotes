# OpenGL
OpenGL is a **specification** that has implementations across different graphics cards that are maintained by the producers. 

**Immediate Mode:** Referred to as the **fixed function pipeline** -> easy way to draw graphics, but functionality is hidden
**Core-Profile Mode:** More low-level flexibility and more control

Generally, we don't need to use latest version of OpenGL, since only the most modern cards will support it

**Extensions** can be implemented in drivers and used. Developers need to query whether the extension is available before using

OpenGL can be thought of as a **State Machine**, where a collection of variables define how it operates (the **context**)

OpenGL libraries are written in **C** but it supports **objects** -> collection of options that represent a subset of OpenGL's state
