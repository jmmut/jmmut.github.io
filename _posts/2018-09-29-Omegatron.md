---
tags: programming
---

# Omegatron
#### Tron motorcycle game tribute

<iframe
width="560" height="315"
src="https://www.youtube-nocookie.com/embed/7PVa1jbLogU?rel=0"
frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen
allow="autoplay; encrypted-media"
class="lazyload">
</iframe>

This game was initially designed in 2013 as an exploratory project of low level
game engines based on OpenGL and GLUT, programed entirely from scratch. Later,
the game evolved into a decently finished product, ported natively to Android
using the Android NDK.

## Graphic engine

All the graphic engine was build based directly on OpenGL 1. An ad-hoc graph
scene contained all the elements to be rendered. No shaders were used for this
project.

## Physics engine

This engine was one of the most complex parts of the game. Detecting collisions
in a reasonable amount of time was a challenge. A 3D hash map of voxels was used
to register all the elements in the scene. 

In particular, a 3D bresenhan algorithm was developed to choose in which voxels
should the trails be registered.

## Bots AI

Making use of the physics engine, the bots could detect potential collisions,
and use basic heuristics to decide how to escape to open areas.

## Port to Android

Many changes had to be made to the code to make the port. First, as SDL
2.0 was not yet available, we had to rewrite the event handler module to read
the native events from Android. We had to change the input from keyboard to
touchscreen. Also, add a minimal GUI, as the configuration via CLI was not
possible. Then, the OpenGL library had to be replaced with OpenGL ES, with a much
lower level API. The main model, a scooter, was never ported.

Finally, the game was published in the Google Play store. At the time of writing
it is still available.

