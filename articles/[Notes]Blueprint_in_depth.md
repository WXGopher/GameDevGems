Adapted from "Blueprint in depth": https://www.youtube.com/watch?v=j6mskTgL7kU

### Introduction

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210523163730970.png)



Blueprint features

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210524094557436.png)

BP features

1.  inheritance
2.  framework class
3.  components
4.  child actors
5.  library
6.  interfaces
7.  data table/curves/data assets 

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210524100540349.png)



### Performance 

A classical (and bad and complicated) example:

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210524110110641.png)

Keep everything as simple as possible but have enough complexity.

Here is a remark on the performance of BP:

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210526201023517.png)

Performance is mainly cost in between BP nodes (not within BP nodes themselves). This only happens in execution (not itself, say, embedding content)

So do not abuse

1.  quick loops
2.  get a large number of actors/classes 
3.  **ticking** (most common)

More on ticking:

Ticking does not always mean event ticking, it also happens in Anim BP or even in UMG. Here are something you can do to mitigate ticking impacts:

1.  You can set whether BP ticks by default in engine settings, or you can set if a BP ticks and its ticking frequency in its setting (or in some BP nodes, like `Set Actor Tick Interval`)
2.  You can also use other events or even a timer to replace an event ticking.
3.  You can separate physics and rendering and use a timer for physics (if possible)
4.  You can dispatch some computations to material (on GPU)

BP is currently always executed single-threaded.

BP executes in a difference speed under different scenario (PIE, dev play, shipping play), like this experiment:

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210526201905278.png)

Meanwhile native (C++) is blazingly fast (would be the x-axis if shown on the diagram above).

BP **majorly impact** memory consumption and loading times. It is regard as content while loading. 

C++ are loaded on project boot but contents are loaded on an as-needed basis. Unless that content is loaded with C++ classes that reference it (loaded on boot).

*Any form of referencing* (including casts) between BP and another BP or content will load it. 

So avoid loading a BP that references a loooot of other BPs or content, instead split those off into child BP classes, in a word:

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210526205717391.png)

and an example:

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210526205806465.png)



### Compilation

Quick remarks on compilation:

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210526212913693.png)

There IS limitation on BP compilation:

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210526213025637.png)

To speed up compilation:

1.  use less nodes 
2.  use less cast (cast is a reference and will include another class). You can also use tag to function as a type check usually handled by cast
3.  function reduces compiling time significantly
4.  computational expensive job should be moved to C++



### Misc.

Ideally in larger projects, most classes should at all times be made in C++ but extended in BP. 

![](https://raw.githubusercontent.com/WXGopher/GameDevGems/master/images/UE4_bp.assets/image-20210529095638581.png)