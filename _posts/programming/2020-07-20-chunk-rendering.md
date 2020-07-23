---
layout: post
title:  "Chunk rendering in a minecraft-like game engine"
date:   2020-07-20
desc: "My experience on optimizing the rendering of chunks in a minecraft-like engine using OpenGL ES 3.0 and SDL2."
keywords: "Canas,Simon,gh-pages,website,blog,engine,chunk,minecraft"
categories: [Programming]
tags: [Programming,Engine]
icon: icon-cplusplus 
---

For the end of my second year of study, my classmates and I have all been tasked on working on a small Minecraft-like project using the NekoEngine, a SDL2 based 3D game engine using OpenGL ES 3.0 developed by our teacher (<a href="https://eliasfarhan.ch/">Elias Farhan</a>) that we've been working on for several months.

<div class="navy-line-no-margin"></div>

# The Problem
The most important part was obviously to create a solid chunk structure, since it became clear quite fast that stocking each block individually in the chunk would fill up the memory by an insane amount, and rendering more than 100 cubes at a time would be impossible which is obviously a no-go.

The first thing I did was implementing instancing into my rendering and the results were immediate, but it wasn't enough since it could only display one type of block for the whole chunk.

After some brainstorming and help from my teacher, I managed to create a new structure that would run smoothly but also allow multiple types of blocks to be rendered.

# The Actual Chunk Structure
A chunk is an entity that can hold 16x16x16 blocks that has a total of 4 components:

<center>
	<img src="chunk-struct.webp" width="70%"> <br>
	The component structure of a single chunk.
</center> <br>

For this blog post, only the **CHUNK_CONTENT** and **CHUNK_RENDER** components are important.

<div class="navy-line-no-margin"></div>
# The Block Manager
The **BlockManager** is a single class that is here to holds the various data of blocks:

<center>
	<img src="block-manager.webp" width="70%"> <br>
	The class structure of the BlockManager.
</center> <br>

As you can see all the blocks are stored once in a vector, which can be then retrieved from there. The important element here is the **BlockTex** struct that holds the indices of the textures of the side, top and bottom faces. If **topTex** or **bottomTex** are equal to zero, the side texture will be used instead.

Here, the definition of a "TextureId" isn't the same as in OpenGL. Every textures are stored on a single, larger texture called "atlas", which you can see a snippet of here: 

<center>
	<img src="atlas.webp" width="50%" class="pixelated"> <br>
	A snippet of the altas whith the texture indices.
</center> <br>

As you can see, the texture index corresponds to its position on the atlas, this is useful since it makes that many textures that we don't need to load and pass the fragment shader.

# Chunk Render
This is a simple component that holds one **VBO** used by the ChunkContent and a **RenderCuboid** which is just a cube with the vertices positions, normal, etc...
There isn't much special about this component, so there isn't really anything more to say about it.

# Chunk Content
The **CHUNK_CONTENT** component consists of a vector (ugly, I know) of **ChunkContent**: a small struct that only holds 2 values, **blockPosId** and **texHash**, the first being a hash of the cube position and the second a hash of the **BlockTex** struct that can be retrieved from the BlockManager. This graph will make it simpler to understand how that works:

<center>
	<img src="chunk-content.webp" width="80%" class="pixelated"> <br>
	How the ChunkContent component is created and how its values are passed to and processed by the shaders.
</center> <br>

The resulting **ChunkContent** from this operation is then given to the vertex shader using the ChunkRender VBO, where it calculates the correct texture coordinates from the texHash and give those to the fragment shader. The blockId will then be used to generate the model matrix, and since we don't need to rotate nor scale the block it is a pretty simple operation.

All of those operations allows us to only give the minimal amount of data to the shaders as well as allowing different block types using instancing.

# The result
<center>
	<img src="result.webp" width="80%" class="pixelated"> <br>
	Compiled with MSVC v16.5.
</center> <br>

Thanks to all this optimization, I am able to render 169 completely filled chunks (4096 blocks per chunks), without using any performance saving technique like frustum or occlusion culling, at a stable 70 fps on Windows 10, with an Intel i7-4790K@4.00GHZ CPU and a GeForce GTX 970 GPU.


# Conclusion
I learned several tricks during this project, notably how to reduce the memory usage on the GPU and how to squeeze every bit of performance out of it.

This has also been quite an enriching experience in teamwork, since I had to organize my work around what my other classmates has done.

