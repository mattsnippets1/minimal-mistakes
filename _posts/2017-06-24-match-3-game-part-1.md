---
title:  "Match 3 game Part 1 - Basic match 3 implementation"
date:   2017-06-24 13:00:00 +0200
tags: Match3
---
In the last two weeks I have started another project - the implementation of a 2D match 3 game similar to Bejeweled or Candy Crush Saga.
<!--more-->

<iframe width="560" height="315" src="https://www.youtube.com/embed/xk_StqpgBSI?rel=0" frameborder="0" allowfullscreen></iframe><br>
I have successfully written a barebone implementation with basic functionality: piece switching and matching via mouse drag, column collapse and a refilling board. Of course there's tons of stuff still missing: score counter, special game pieces, levels etc. I would also like to generate a mobile (and possibly a web player) build as well.

I tried to organize the code into separate components and I'm not 100% satisfied with the results so far but it's not a single monolithic class at least :D.

The board gameobject consists of tiles and each tile contains a gamepiece from six different colours. The board has a script attached to it (which is called - surprise - **Board**). It has some parameters configurable via the editor (width, height, border of play area, switch speed). It also runs the main game logic: setting up the board by initializing all **Tile** objects and filling it with random game pieces is managed by this script along with the game piece switching which is triggered via mouse events.

At the start of the scene the board is generated such that it contains no matches. This is achieved by the **FillAtStart()** method which generates the pieces one by one and then checks if the piece creates a match in its row or column. If it does, the piece is removed and another random piece is generated instead. A pool could have been added to the random generator but I counted the iterations and there's no significant difference (not with a 9x9 board at least).

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

private void FillAtStart()
{
    for (int i = 0; i < width; i++)
    {
        for (int j = 0; j < height; j++)
        {
            if (i == 0 && j == 0)
            {
                pieceManager.CreateRandomPiece(i, j);
            }
            else
            {
                do
                {
                    pieceManager.RemovePiece(i, j);
                    pieceManager.CreateRandomPiece(i, j);
                } while (matchFinder.FindMatchesAt(i, j).Any());
            }
        }
    }

    Debug.Assert(!matchFinder.FindAllMatches().Any(), "There are matches in the start board!");
}

{% endhighlight %}

**Board** has other objects attached to it which handle different logical components: **CameraManager**, **MatchFinder**, **BoardInputHandler**, **ColumnManager** and **PieceManager**.

**CameraManager** configures the position and ortographic size of the camera based on the gameboard dimensions. **BoardInputHandler** sets the value of a **selectedTile** and a **targetTile** variable upon click and drag. When the player releases the mouse button and theres a start and target tile set, an **Action** callback is invoked from **BoardInputHandler** which triggers **SwitchPiecesCoroutine()** in **Board**. Only adjacent pieces can be dragged on each other and a check is run each time to ensure a new match is formed with the switch. If there's no match, the switch is reversed and both pieces are moved back to their original coordinates.

**MatchFinder** is the class responsible for finding matches between pieces. It has two public methods: **FindMatchesAt()** checks a certain location for matches while **FindAllMatches()** searches the whole board for matches. Three or more pieces of the same colour in a row or column is considered a match, there's no diagonal matching.

**PieceManager** handles the 2D array containing all **Piece** object references. This class can create, place and remove pieces. The piece array is encapsulated within a wrapper class to avoid changing the value of an array element by accident. Therefore, the wrapper's indexer is readonly and array elements can be only assigned through a setter method. Unfortunately, that doesn't mean that the objects within the array are protected I'm only using this approach to avoid accidentally overwriting an array element from another class.

**ColumnManager** creates the sliding effect which is seen when pieces are removed from the board. Currently there's no collapse for the new pieces generated after a match but I'm planning to introduce that as well. **SlideColumnAt()** is invoked on all the columns with pieces removed. The method searches for null pieces in the column then iterates through all of the pieces above to find the next non-empty piece which is moved down to replace the empty game piece. This process is repeated until all the non-null pieces fall into place.

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

private List<Piece> SlideColumnAt(int columnIndex)
{
    List<Piece> piecesToMove = new List<Piece>();

    for (int i = 0; i < height - 1; i++)
    {
        if (pieces[columnIndex, i] == null)
        {
            for (int j = i + 1; j < height; j++)
            {
                if (pieces[columnIndex, j] != null)
                {
                    pieces[columnIndex, j].Move(columnIndex, i, 0.1f * (j - i));
                    pieces.SetElement(columnIndex, i, pieces[columnIndex, j]);
                    pieces[columnIndex, i].SetCoordinates(columnIndex, i);

                    if (!piecesToMove.Contains(pieces[columnIndex, i]))
                    {
                        piecesToMove.Add(pieces[columnIndex, i]);
                    }

                    pieces.SetElement(columnIndex, j, null);

                    break;
                }
            }
        }
    }

    return piecesToMove;
}    

{% endhighlight %}

The **Piece** and **Tile** classes are quite simple, they contain coordinates, game piece type and similar stuff. **Tile** has some mouse event handlers to detect player interaction and **Piece** manages its own movement with a lerp.

The simplified flow of the game is like this:
1. Player clicks on a piece and drags it towards neighbour
2. Upon release switch check is run
3. If a switch is possible **ClearRefillCoroutine()** is invoked which runs **ClearCoroutine()**
4. **ClearCoroutine** is invoked recursively, removing all matches
5. **ClearRefillCoroutine()** invokes a method filling all empty tiles
6. The board is checked again for matches. If there are any, the process is repeated from 3.

I won't paste most of the code here as it is much more than a few snippets but you might find all of it on my BitBucket. I used [Jelly Squash Free Sprites](http://www.gameart2d.com/jelly-squash-free-sprites.html) from gameart2d.com.