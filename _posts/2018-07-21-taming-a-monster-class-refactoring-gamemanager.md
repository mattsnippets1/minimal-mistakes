---
title:  "Taming a monster class - Refactoring GameManager"
date:   2018-07-21 16:00:00 +0200
tags: refactoring gameplay
---
Despite trying to avoid it, my code has fallen victim to the fearsome **GameManager** class. Here's how I managed to fix it (or make it better at least).
<!--more-->

If you read a bit about organizing code and designing your classes in Unity (or other writings on the topic of game code design) you might encounter the advice to avoid creating omnipotent "god classes" in your game. Most of the time these classes are called "...manager" and they tend to become bloated and unmanageable.

They clearly violate the proposed Single Responsibility Principle of OOP and while I think recklessly forcing object oriented design on Unity is not a good idea, it might still be useful to consider what tasks your scripts are performing and if they are organized in a practical way.

When I started working on Corpse Mob I also decided to create a **GameManager**. There were some parts of business logic which I could not fit into other classes and while most of the GameObjects set their initial states I still had to make sure that some of the components are initialized in a given order. At first it seemed quite innocent, a small class maybe a bit over a hundred lines.

Unfortunately as the time of release was getting closer and closer I found myself throwing some more code in **GameManager** which I would "sort out later". Of course, that didn't happen, at least not until the past few weeks. I came to the decision that I have to refactor and separate most of the manager class' code if I want it to be maintainable in the future.

The first step was to cut **GameManager**'s methods into smaller parts and give them descriptive names. This made it much easier to collect all the methods within the script that could be working as separate logical units and it was easier to see what was "out of place". Skimming through the reorganized code it was obvious that GameManager had wrapped its tentacles around many parts of the game logic, including but not limited to:

* Managing boss fights (victory conditions, initializing special cases of boss monsters, boss health GUI)
* Gathering and saving data for gameplay statistics (e.g. amount of zombies killed, number of kills with certain weapons)
* Managing high scores and sending a new high score to Steam leaderboards
* Initializing wave data (target score to complete a wave, monster spawn settings)
* Handling the number and usage of monster decoy items
* Handling player level up processes
* Initializing the state of player stat classes and their observers to follow the change of a stat value
* Managing timescale changes (bullet time, pausing and unpausing)

I moved some of these responsibilities to existing classes, like a list of decoys which only **MobSpawner** used found its new home in **MobSpawner** code or some initialization logic could be reorganized to the **Weapon** and **WeaponManager** classes themselves, using their **Awake()** and **Start()** methods to remove a lot of coupling between components. Data objects containing wave settings have been rewritten as ScriptableObjects and are now referenced from the manager.

Other pieces of code have been refactored into brand new classes like **BossfightHandler** or **PlayerLevelManager**. Yeah, these classes might also sound suspicious because I wanted to eliminate the manager class in the first place but they are smaller (mostly between 100-200 lines), do one thing and they are modular.

The monster class which was more than 900 lines long has been cut back to around 500. Also much of its length now comes from the numerous smaller methods which are used for readability of code. Not a bad start, I think.

![Splitting parts of GameManager into new and existing classes]({{site.url}}/assets/images/gamemanager-split.png){: .align-center}
*Splitting parts of GameManager into new and existing classes*
{: .text-center}
