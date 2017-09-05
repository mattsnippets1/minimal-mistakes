---
title:  "Match 3 game Part 5 - Move and time limit, bugfixes, level select, move hint, particles, new assets"
date:   2017-07-31 20:00:00 +0200
tags: match3 GUI
---
Some issues had to be fixed, new GUI functionality was added to the game along with move hints, move and time limits and particle effects.
<!--more-->

While winning the game was already possible by reaching the target score I wanted to add losing conditions too. By modifying **GameManager** I added move and time limits to the game. Time limit is managed in **GameManager**'s **Update()** method: deltaTime is subtracted from the **remainingTime** field's value, if the remaining value is below 0 a **GameOverLose** message is sent from **GameManager** otherwise **GuiManager** is ordered to update the remaining time text.

For the move limit there is a **ValidMove** event which is handled in **GameManager**'s **OnValidMove()**. Here **remainingMoves** value is decremented and game over and GUI updates happen in a similar fashion to time limit.

**GuiManager** has become more generic with a **SetTextValue()** method which finds a text field by name and sets its value. There's also a **DisableTextElement()** method based on the same idea. Separate event handlers were added for winning and losing.

With a slight modification to **Board** the movement of pieces is now disabled on game over so no pieces can be moved after the game over dialog has appeared. There was another issue with color bombs considered as matching pieces. An extra check was introduced to **MatchFinder** so color bombs cannot match anymore.

After creating a bunch of levels with different winning and losing conditions it was logical to make a level selection screen. Currently it is nothing more than a few buttons on a tiled background, each of them loading the corresponding level (scene). There's also a home button in every level to let the player go back to level select.

![Level select screen]({{site.url}}/assets/images/match3-level-select.PNG){: .align-center}
*Level select screen*
{: .text-center}

One of my next goals was the implementation of a hint marker to indicate a possible move for the player. If there are no more valid moves on the board it should lead to a game over. As a temporary solution I used **AutoTester**'s **FindMove()** method. It had already been used to find a possible move for the automatic piece moves. Now this functionality is also invoked in **Board**'s **ClearRefillCoroutine()**. Upon finding no matches a **GameOverNoMatches** message is broadcast by **AutoTester**.

The next step was the hint marker itself, created as a "sparkle" effect with the use of particles. **moveHint** is a **CoordinatePair** field in **Board** and the return value of **FindMove()** is assigned to it in **Start()** as well as at the end of **ClearRefillCoroutine()**. In **Board**'s **Update()** the time since last move is measured and **SetHintMarker()** is called. If the timeout for hints elapsed the markers are activated (or instantiated for the first time) and their position is set according to **moveHint**.

Another particle effect was added **PieceManager**'s **RemovePiece()** so pieces now explode when removed. I added a **SerializableDictionary** field to **PieceManager** where the keys are strings and values are Unity colors so explosion colors can be matched with piece types easily. The piece explosion prefab has an attached script which destroys the whole GameObject after a few seconds.

Finally, most of the assets have been revamped, new piece graphics have been added and the borders of the tiles have been replaced with a checker pattern.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xYI8sDIQTNo?rel=0" frameborder="0" allowfullscreen></iframe><br>