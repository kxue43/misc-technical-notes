OpenGL and Vulkan
=====================

The relationship among graphics card, graphics card driver, OpenGL/Vulkan and widget toolkit (GTK, GT) is like the
relationship among database engine, PEP-249 package, SQLAlchemy and packages that might conditionally use SQLAlchemy.

Graphics card drivers are kernel modules that interfaces with the underlying hardware. As there are many different
drivers, there's need for a uniformization layer, which is OpenGL and Vulkan. Not only that, these APIs are usually
used to interact with GPU to achieve *hardware-accelerated rendering*.

Both OpenGL and Vulkan are API specifications. OpenGL is easier to use. Vulkan exposes more low-level operations and
is more powerful for programmers that know how to use it. The same names are probably also used to refer to some widely
used implementations of these APIs. Nonetheless, there are more than one implementations. For example,
Apple's Metal API and Microsoft's Direct3D 12 are both compatible with Vulkan. On the other hand, Mesa 3D is
"an open source implementation of OpenGL, Vulkan, and other graphics API specifications".

GTK and QT provide library code for creating GUI widgets, but how these widgets get rendered on the screen probably
depends on how the code is compiled. It might also be possible to compile the code so that at runtime the program
probes the OS for available options and choose the right one. Bottomline, both GTK and QT can render with OpenGL,
maybe Vulkan as well.
