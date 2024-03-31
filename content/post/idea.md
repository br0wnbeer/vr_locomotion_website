+++
title = 'Idea'
date = 2024-03-30T19:33:41+01:00
draft = false
summary = "main idea"
+++
# Introductioin
The Idea to create a locomotion and interaction method came form my Univerestey course (Interaction in VR and AR). They tasked us to implement such  methods in Unity in VR using the Meta Quest 2. 
# My Locomotion Idea
My Idea was it to implemt a locomotion technique the was inspired by the locomotion teqnique that was intorduced in [Team Fortress II](https://www.teamfortress.com/) by a class called the [*Soldier*](https://wiki.teamfortress.com/wiki/Soldier). The locomotion teqnique involes the charecter moving by launching rockets so that the explosion force moves the carater itself. An example of this kind of locomotion can be seen [here](https://www.youtube.com/watch?v=fsf0M-lzGUo).
{{< youtube fsf0M-lzGUo >}}
For my implementation I wanted to reduce the amount of vection that is intorduced to the player, trying to prefent cyber sickness. If I implemented my method in the same way it was, the vection would have been verry high. For this I came up with a few ideas to prevent/lower it: 

* reducing the amount of push each explosion gives to player 
* inorducing some wind to the player to give a perception of motion 
* giving virtial and audio queues that the player is moving 

Do to some limitations of course not all of these were able to be intorduced in the final game. 

# My Interaction Idea 

Trying to stay in the theme of rockes I wanted to also have some other kind of projectile to be used in my interaction method. For this I chose what I call *Grabbing Boomerang*. Here we have a projectile that collides with interactable objects and if possible bring them to the players hand so that the player can roate as well as do a translation in the world space.
