+++
title = 'Idea'
date = 2024-03-30T19:33:41+01:00
draft = false
summary = "Main Concept"
+++
# Introduction
The Idea to create a locomotion and interaction method came form my University course (Interaction in VR and AR). They tasked us to implement such  methods in Unity in VR using the Meta Quest 2. 
# My Locomotion Idea
My Idea was it to implement  a locomotion technique the was inspired by the locomotion technique that was introduced in [Team Fortress II](https://www.teamfortress.com/) by a class called the [*Soldier*](https://wiki.teamfortress.com/wiki/Soldier). This technique involves , the character moving by launching rockets so that the explosion force moves the character itself. An example of this kind of locomotion can be found  [here](https://www.youtube.com/watch?v=fsf0M-lzGUo).
{{< youtube fsf0M-lzGUo >}}
For my implementation I wanted to reduce the amount of vection that is introduced to the player, trying to prevent cyber sickness. 
If I implemented the locomotion in the same way it was presented in Team Fortress II, the perceived motion would have been verry high. To prevent this I came up with a few ideas to prevent/lower it: 

* reducing the amount of push each explosion gives to player 
* introducing some wind to the player to give a perception of motion in real life
* giving virtual and audio queues that the player is moving 

Do to some limitations of course not all of these were able to be included in the final game. 

# My Interaction Idea 

Trying to stay in the theme of projectiles I wanted to also have some other kind of projectile to be used in my interaction method. For this I chose what I call *Grabbing Boomerang*. Here we have a projectile that collides with intractable objects and if possible bring them to the players hand so that the player can rotate as well as do a translation in the world space.
