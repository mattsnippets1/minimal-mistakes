---
title:  "Corpse Mob 1.1 patch"
date:   2018-08-09 20:00:00 +0200
tags: corpsemob
---
Today I have released the first major Corpse Mob update (version 1.1) on Steam.
<!--more-->

This patch containts the refactored code I've been working on for the past few months. It also introduces lots of bugfixes (and not so many new bugs, hopefully), gameplay and balance tweaks and some post-processing effects to buff the visuals a little.

Complete list of changes for those who love lists:
- Post-processing visual effects such as bloom and color correction added
- Added possibility to turn off screen shake effect in options
- Reduced default volume when game is started for the first time
- Added circular HP bars
- Waves have become shorter, with lower score targets
- Tweaked level up points amount depending on number of players
- Lots of refactoring done in code, making it easier to be maintained and extended
- Fixed the order of muzzle fire and bullet sprites in some cases
- Fixed some Z-fighting among dead zombie sprites
- Switched from gamma color space to linear
- Changed blood splat shaders, made them bigger
- Fixed stuck chainsaw sound
- Changed the appearance of gas clouds
- Fixed a bug with player disappearing when revived right after dying in co-op
- Added D-pad support in character selection menu
- Fixed a bug that wouldn't let the game unpause
- Added E as place decoy button when using keyboard controls
- Tweaked bullet hit blood particle settings
- Removed player "level up target score system"
- Arrow indicating another player down rotates when body is at the top of the screen
- Lowered boss fight unlock kill targets