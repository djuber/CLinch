# CLinch

## Overview

CLinch is a simple, yet powerful 3d graphics engine for Lisp. In true Lisp fashion, it is more a build-your-own-engine hobby kit rather than striving to be a full graphics panacea. Much of the design is based on Horde3D. Although Horde3D is a very good design, it has limitations which it attempts to solve by adding domain specific languages. Items such as shaders, entities and, most importantly, the pipeline are specified using an XML language. Lisp, by being self-compiling seemed like a better fit than requiring (yet) another language.

Eventually I hope that CLinch becomes a stable and fast workhorse tool for developing games, visualizations and productivity software. I have plans for a graphical shell which incorporates the strengths of Lisp and 3D. See my [Qix](https://github.com/BradWBeer/Qix) project for more information. I also want CLinch to become a base for live-coding games and demos. 

### CLinch Extensions:

* Integration with 2D vector graphics with Cairo

* Integration with fonts and text positioning with Pango

* File loading and saving with FreeImage


## Design Goals

### Fast 

CLinch should be as fast or faster than most script-based graphics engines while requiring much less development time. While it may never rival professional C libraries, the ability to modify the 3D engine and environment from the REPL will allow skilled developers to create applications in a fast, intuitive, flexible way.

### Simple

CLinch should be as simple as possible for someone familiar with 3D graphics programming to understand. I still remember how easy it was to write a single pixel to the screen in DOS and while I can't simplify to that degree (without losing modern power and flexibility), I can remove many of the most common difficulties. These include texture and vertex buffers, shader compiling and linking, shader variable passing, drawing text and 2D graphics, pipelines and 3D transformations. 

### Intuitive

Time spent looking for solutions is time spent, period. It ruins developer flow and can stop a project (especially a small one) indefinitely. CLinch should have few basic primitives which solve the general cases well and allow for easy replacements when necessary. It should be well documented and have a consistent interface. I will strive to keep abstraction leaks at a minimum. 

### Flexible

CLinch should not depend on any one library. Currently it only supports OpenGL, but the framework can (in theory) support others. It should not rely on a particular windowing library and should work as a simplified replacement for the original graphics library interface. It should not be designed for any one type of application such as games, but should support any 2D or 3D application. It should be cross-platform (wherever possible) and not inflict any particular design onto its client applications.


## Architecture

Although CLinch can be used as a complete graphics engine, most parts of CLinch are independent. You can use most objects by themselves as best suits your application. For example, you can use a buffer object by itself. This also helps with testing by isolating the various parts of CLinch. The following is in hierarchical order based on the default configuration. This is to explain it as clearly as possible, not to indicate a necessary design. 

Most objects have children method which tries to behavior intelligently based on their type. Most objects also have a render method which will render itself then its children.

### Transforms

A transform is a 4x4 matrix which is used to hold and apply a C array of 16 floating values. Currently only single-floats are supported. It encapsulates the CFFI memory management using trivial-garbage. It also has basic arithmetic operations for 4x4 linear algebra and 4x1 vectors. (Note: only m#v is currently implemented) There are functions for rotate, translate, scale, multiply (m#), make-identity (or reset), transpose, determinate, invert, make-orthogonal-transform, make-frustum-transform, make-perspective-transform, multiply by vector (m#v), use-transform (for the OpenGL matrix stack) and get-current-gl-matrix. Modern transform functions for passing values to a shader are not yet supported, but should be soon.

### Nodes

Nodes are not usually the topmost objects in the hierarchy, however they are the most important. Nodes abstract 3D transform "chains" and hierarchy. Nodes subclass transforms and can be modified the same way as transforms. They also may have children. Nodes are multiplied together in a hierarchy to create an effective transform which it stores in a hash by a list of all its parents. This allows a node to be reused in any way necessary. A node can be used in multiple places within a hierarchy (or tree) or even in several different ones. This is done by passing a list of parents to the update function. It will then append itself and call update on its children. In this way, the transforms are only calculated when they or their parent(s) have changed. 

### Buffers

Buffers abstract the shared data on the graphics card. They can carry almost any data to the graphics card in bulk including vertexes, normals, and texture coordinates. Buffers are usually sent to a shader using the entity object. A buffer's data can be accessed directly by using the map/unmap functions, or more simply using with-mapped-buffer.

### Textures

A subclass of Buffer, textures abstract the 2D textures and allow easy access to their raw data. Textures have all the functionality of buffers but also have width, height and data format. Since it is a subclass of Buffer, use map/unmap or with-mapped-BUFFER.

### Shaders

Shaders are the compiled output from text-based vertex and fragment shaders. They require a name, vertex shader source code, fragment shader source code, a list of uniforms and a list of attributes. Vertices and index buffers need not be specified if you are using them. 

### Entities

Entities are what are rendered and are also a subclass of node. They bring together the shader, buffers, textures, attributes and uniforms and transforms together into something which can be rendered on the screen. A shader and a (currently) an index buffer is required although they would be useless without at least one vertex buffer. Each member of values can be either an :attribute, :uniform or :vertices. 

### Viewports

An application can have several viewports. As CLinch does not force any particular windowing implementation, it can not have a window class. Once your window is set up, however, you can use the viewport handle drawing an area on screen. The most common children the cameras, which will be rendered in order unless then are not enabled.

### Pipelines

A pipeline is a series of commands which create the scene with every call. Generally they clear the screen, render the root node and sorting items into the proper sequence. Unlike Horde3D, any set of Lisp can be used. Usually there is only one pipeline per viewport or application.

