---
title:  "Match 3 game Part 3 - Bomb spawning, testing"
date:   2017-07-18 20:00:00 +0200
tags: match3 testing
---
I have added bomb spawning logic to the code and also created some test solutions (using an automatic tester class and Unity Test Tools).
<!--more-->

Until now bombs could only be added to game pieces manually in the editor but I wanted to make them appear on matches of 4 or more pieces as it's required for real gameplay. **PieceManager** got extended with some new methods: **AttachBomb()**, **IsCornerMatch()** and **CreateBombData()**.

Whenever a match is made during a switch it is analyzed by **CreateBombData()**. If the piece count in the match is greater than or equal to 4, a **BombData** instance is returned. **BombData** stores the information which is used a bit later in **ClearRefillCoroutine()** to create and attach bombs to the game pieces involved. **IsCornerMatch()** is used to decide the type of the new bomb (block bombs are created when pieces are matched in a T or L shape, otherwise a column or row bomb is created depending on the direction of the matching move).   

{% highlight c# %}

public void AttachBomb(int x, int y, BombType bombType)
{
    Assert.IsTrue(Board.IsWithinBoard(x, y));

    Transform parentPiece = pieces[x, y].gameObject.transform;
    GameObject attachedBomb = Instantiate(bombPrefab,
                        Vector3.zero, Quaternion.identity) as GameObject;
    attachedBomb.transform.parent = parentPiece;
    attachedBomb.transform.position = parentPiece.position;
    attachedBomb.GetComponent<Bomb>().Init(bombType);
}

public bool IsCornerMatch(IEnumerable<Piece> pieces)
{
    bool isVertical = false;
    bool isHorizontal = false;
    int startX = -1;
    int startY = -1;

    foreach (Piece piece in pieces)
    {
        if (piece != null)
        {
            if (startX == -1 || startY == -1)
            {
                startX = piece.X;
                startY = piece.Y;
                continue;
            }

            if (piece.X != startX && piece.Y == startY)
            {
                isHorizontal = true;
            }

            if (piece.X == startX && piece.Y != startY)
            {
                isVertical = true;
            }
        }
    }

    return (isHorizontal && isVertical);
}

public BombData CreateBombData(int x, int y, Vector2 switchDirection, IEnumerable<Piece> pieces)
{
    BombData bombData = null;

    if(pieces.Count() >= 4)
    {
        if(IsCornerMatch(pieces))
        {
            bombData = new BombData(x, y, BombType.Block);
        }
        else
        {
            if (switchDirection.y != 0)
            {
                bombData = new BombData(x, y, BombType.Row);
            }
            else
            {
                bombData = new BombData(x, y, BombType.Column);
            }
        }
    }        

    return bombData;
}

{% endhighlight %}


My other goal for the last two weeks was the implementation of some test framework for the match 3 game. My first idea was the creation of an **AutoTester** class. An instance of this class is created from the **Board** script. It is initialized with most of the important components of **Board** e.g. the tiles array, width and height and also some other components: **PieceManager**, **BoardInputHandler** and **MatchFinder**. **AutoTester** functions as a proxy for other test scripts (data from **Board** might be acquired through **AutoTester**) and it can also "play" the game by itself.

**AutoTester**'s **TestMoveCoroutine()** runs in an infinite loop while there are any valid moves on the gameboard. It invokes **FindMove()** which in turn creates a copy of the current piece array wrapper, finds all the possible moves in it by trying to swap the pieces with their right and upper neighbours and puts all of them in a list of **CoordinatePair**s.

A possible move is chosen randomly from the list and it is also performed by manipulating **BoardInputHandler**. If there are no more moves the algorithm stops with a debug warning. **AutoTester** can be run by ticking a checkbox in the editor.

While **AutoTester** is good for simulating many hours of gameplay I also tried to exploit the possibilities of the recently integrated Test Runner and Unity Test Tools which can be downloaded as a separate asset.

Unit tests can be implemented in Test Runner and while it is really sad that it lacks the possibility to measure test coverage at least it's free from the usual security exceptions encountered while using third party unit test frameworks for MonoBehaviours. I wrote some unit tests for **MatchFinder** and **PieceManager**.

I also created some integration tests using Unity Test Tools. These are supposed to help in the validation of game board initialization, bomb spawning, explosion logic and also basic gamepiece switching (testing functionality which cannot be invoked via unit tests).
