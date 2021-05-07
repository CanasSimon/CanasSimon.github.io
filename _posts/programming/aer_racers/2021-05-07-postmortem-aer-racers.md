---
layout: post
title:  "Postmortem: Aer Racers"
date:   2021-05-07
desc: "A retrospective look at my work on our game project Aer Racers."
keywords: "Canas,Simon,gh-pages,website,blog,engine,aer,racers,game"
categories: [Programming]
tags: [Programming,Engine]
icon: icon-cplusplus 
---

During the first half of my final year of study, my classmates and I were tasked to work together on a game project. We have now worked on it for 9 months, finally reaching its end.

The game was a racing game named Aer Racers created using the <a href="https://github.com/EliasFarhan/NekoEngine">NekoEngine</a>, and was for our entire team the first time we had to make a project of this scale. We were under the supervision of our teacher, <a href="https://eliasfarhan.ch/">Elias Farhan</a>, during the entire project, but he generally let us handle the project on our own, only intervening when things got out of control.

<center>
	<img src="aer-racers.webp" height="200px"> <br>
	The logo of our game.
</center> <br>

Each one of us had different roles to fulfill, and I was appointed as the Lead Engine Programmer.

<div class="navy-line-no-margin"></div>
# Creating a Vulkan graphics engine
The very first task I was given was the creation of a Vulkan graphics engine alongside the already existing OpenGL one. However, since I had never worked with Vulkan before so I had to begin learning Vulkan and make the engine from scratch using the already existing structure of the NekoEngine.

This singular task was a constant red string during the entirety of the project, constantly being upgraded and worked on as the project continued. On top of that, I also had to maintain the already existing OpenGL engine that we had in order to have at least a backup plan in case the Vulkan part ended up not being usable.

During the first months, the people responsible for developing the game were making a prototype in Unity to be able to work efficiently which left me a lot of breathing space to develop the engine without fearing to step on anyone's foot. I chose to first work on it on a seperate environment in order to slowly, but surely learn how Vulkan worked.

The first 4 months were definitely the toughest, since we were such a small team, I was left to work on the Vulkan engine all by myself, and learning it as I went, which definitely brought some headaches. Even getting a 3D object to correctly render took me an absurd amount of time compared to OpenGL.

<center>
	<img src="cube.webp" height="200px"> <br>
	After almost a month, I finally had a texture cube.
</center> <br>

After that, I needed to port what I had made into the NekoEngine and adapt it for a game engine. That was the part I had the most difficulties of all, as what I've done in the separate environment was mostly just some hardcoded blocks that needed to be abstracted. The basic Vulkan setup process was thankfully painless, as there wasn't much to change, but it was as soon as I needed to start loading models that I began to struggle. Loading textures and models by themselves wasn't a very hard task to abstract, as it was mostly all contained in a TextureManager and ModelManager, but the problem came when passing the uniform data to the shaders.

<div class="navy-line-no-margin"></div>
# Vulkan and shaders
As it turns out, vulkan is quite inflexible when it comes to uniform data, while in OpenGL you can just give the name of the uniform and its value, in Vulkan you need to explicitly declare beforehand which uniforms the shader posseses and their properties, like binding, size, type, name, and more. To solve this problem, I ended up creating a json file for each shader which would hold all the information that I needed in the code.

<div class="navy-line-no-margin"></div>
# An unexpected change in task
After struggling some more on the Vulkan engine, I succeeded on rendering multiple 3D textures objects, and was gonna start integrating all of this into a RenderManager to tie the Vulkan engine with out Entity Component System. However, at some point we realized that we needed to create a Showroom to visualize the models for the artists that were gonna join use in a couple of months, and since this was a graphics related task and that it would still take some more time before the Unity-to-NekoEngine transition to start, I decided to take on the task of creating the Showroom.

I ended up making it in OpenGL to speed up the process. The creation process went smoothly, albeit a bit too slowly at times. This also allowed me to experiment with the different shaders and certain effects like bloom and toon shading. In the end I managed to create a tool I was satisfied with:

<center>
	<img src="showroom.webp" height="400px"> <br>
	A screenshot of the finished Showroom.
</center> <br>

You're able to do some simple operation like move the model and the light around, as well as change the model textures if needed. It was functional and created a build and uploaded it on our team shared drive.

However, when the time came for the artists to come and work with us on the project, multiple of them reported that they couldn't launch the program and some others said the shaders looked odd sometimes:

<center>
	<img src="ship.webp" height="300px"> <br>
	The ship looked way paler than it should.
</center> <br>

It was at this point that I completely forgot the basic process of asking for feedback before having the artist work with it. Thankfully, I managed to quickly fix most of the problems that arose, but this was definitely an embarassing experience on my part; these problems simply shouldn't have happened.

<div class="navy-line-no-margin"></div>
# An unexpected departure
Partway while I was working on the showroom, our Lead Tool Programming annouce that he was going to drop out of the course out of personal reasons. This all took us by surprise and we had to quickly find someone to take on his job, and since I was already developing the showroom, which is essentially a tool, I ended taking on his role as well.

Thankfully, at this point in development, there weren't any task on the tool programming side, apart for just polishing up some tools, so overhaul this didn't impact my work load at all, but it became quite hard for the rest of the team that really needed the extra hand that our departed team member could provide.

<div class="navy-line-no-margin"></div>
# RenderManager
Now, at this point in developement, all of the assets were finalized and it was time to create all the necessary systems that we were missing, one of them being the RenderManager.

Our engine uses an entity component system to manipulate objects, so I needed to create a new component that'll allow us to render models at the entities position, that's why I had to create a RenderManager.

It's developement went on smoothly and apart from some minor graphical errors, I didn't encounter any big obstacle during its creation.

I did however struggle quite a bit when implementing the multi-screen in the engine, since our game is supposed to be local multiplayer. The OpenGL part wasn't very hard, since it was just a question of switching viewports and redrawing the scene for each player, but the Vulkan part gave me quite some trouble. The problem was the I couldn't just use the same uniform buffer for all viewports, so I had to create one for each of them, which took me quite some time to figure out, but after some debugging I managed to have a working multi-screen system:

<center>
	<img src="multi-screen.webp" height="300px"> <br>
	4 viewports for 4 players.
</center> <br>

<div class="navy-line-no-margin"></div>
# Conclusion
In the end, even with all of my work, I wasn't able to properly integrate the Vulkan engine into our game, and we ended up using OpenGL, since it was much faster to implement features in OpenGL and we were starting to get short on time. We also encountered some last minute game breaking problems with Vulkan on Windows that I simply hadn't enough time to fix.

Overhaul, this project was a great experience and really allowed me to feel how a game goes from concept to release, albeit in a very condessed form, and much less streeful toward the end. The choice of Vulkan was mostly imposed on us as a learning experience, but, as pointed by almost every one I talk to about the project, Vulkan was by far way too ambitious of a choice for such a small project and ended up eating a lot of my time to not even work at end. Still, I'm glad that I got to work with it as it gave me invaluable experience that'll help me later down the line as Vulkan gets more and more common in games.

My regrets mostly come from my personal investment as I had trouble keeping up multiple times during the project, making me lag behind a bit more than I hoped. On top of that, even though it was a great skill to learn, working with Vulkan got very frustrating multiple times due to me having to learn it as I implemented it in the engine, making me rush through some steps that I would've like to polish. The showroom creation process also really helped hammering down the fact that I need to ask for more feedback when working on programs that is gonna be used by other people.

Even with all the trouble I encountered, this ended being a rather enjoyable project and really helped me find my roots in the engine and graphics programming environment. I still would love to develop a game by myself, but I found myself to be extremely comfortable working in much lower level programming.
