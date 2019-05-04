---
title:  "Match 3 game Part 6 - Particle manager, piece switcher, move finder, refactoring"
date:   2017-08-03 20:00:00 +0200
tags: match3
---
I created a new class for particle management, and two others for switching gamepieces and finding possible moves. The code went through some refactoring rounds.
<!--more-->

I had to fix a minor problem in **MatchFinder** code: color bombs were still considered valid as a starting piece in a match. An extra check solved this issue.

I wasn't really pleased with particle creation code all over the place in **PieceManager** so I wrote a completely new class for this functionality. **ParticleManager** is attached to the **Board** GameObject, can be toggled by a property and has a **CreateExplosion()** method for instantiating explosion prefabs with a given color.   

{% highlight c# %}

using UnityEngine;

public class ParticleManager : MonoBehaviour
{
    [SerializeField]
    private GameObject pieceExplosionPrefab;
    [SerializeField]
    private bool isEnabled = false;

    public bool IsEnabled
    {
        get
        {
            return isEnabled;
        }

        set
        {
            isEnabled = value;
        }
    }

    public void CreateExplosion(int x, int y, Color color)
    {
        if (isEnabled)
        {
            ParticleSystem ps = Instantiate(pieceExplosionPrefab, new Vector3(x, y, 0f),
    Quaternion.identity).GetComponent<ParticleSystem>();

            ParticleSystem.MainModule main = ps.main;
            main.startColor = new ParticleSystem.MinMaxGradient(color);

            ps.Play();

        }
    }
}

{% endhighlight %}

Another addition was the class **PieceSwitcher**. **Board** contained all the logic required for switching pieces which has been moved to this new class. Now **Board** only has a **TrySwitch()** method which calls **PieceSwitcher**'s **SwitchPieces()** coroutine after some tests and a callback method **TrySwitchFinished()** passed as a delegate to **PieceSwitcher**. **PieceSwitcher** itself handles all the logic related to attempting a switch, reverting an unsuccessful one and creating bomb data if the switch should result in bomb creation.

Using **AutoTester** to check for possible moves was a temporary solution and it was time to create a separate class for that task so I wrote **MoveFinder** and moved all the move checking logic to this separate class.   

{% highlight c# %}

using System.Collections.Generic;
using System.Linq;
using UnityEngine;

public class MoveFinder
{
    private int width;
    private int height;
    private PieceManager pieceManager;
    private MatchFinder matchFinder;

    public MoveFinder(int width, int height, PieceManager pieceManager, MatchFinder matchFinder)
    {
        this.width = width;
        this.height = height;
        this.pieceManager = pieceManager;
        this.matchFinder = matchFinder;
    }

    public CoordinatePair FindMove()
    {
        ArrayWrapper<Piece> piecesCopy = new ArrayWrapper<Piece>(pieceManager.PieceArrayWrapper);
        List<CoordinatePair> possibleMoves = new List<CoordinatePair>();

        for (int j = 0; j < height - 1; j++)
        {
            for (int i = 0; i < width - 1; i++)
            {
                if (piecesCopy[i, j] != null)
                {
                    SwitchWithNeighbor(piecesCopy, j, i, 1, 0);
                    CheckMatches(piecesCopy, possibleMoves, j, i, 1, 0);
                    SwitchWithNeighbor(piecesCopy, j, i, 1, 0);


                    SwitchWithNeighbor(piecesCopy, j, i, 0, 1);
                    CheckMatches(piecesCopy, possibleMoves, j, i, 0, 1);
                    SwitchWithNeighbor(piecesCopy, j, i, 0, 1);
                }
            }
        }

        if (!possibleMoves.Any())
        {
            Messenger<string, string>.Broadcast(GameEvent.GameOverNoMatches,
                "No more matches", "No more possible matches on board");

            return new CoordinatePair(-1, -1, -1, -1);
        }

        return possibleMoves[Random.Range(0, possibleMoves.Count)];
    }

    private void CheckMatches(ArrayWrapper<Piece> piecesCopy, List<CoordinatePair> possibleMoves,
        int j, int i, int horizontalModifier, int verticalModifier)
    {
        if (matchFinder.FindMatchesAt(i, j, piecesCopy).Any() || matchFinder.FindMatchesAt(
            i + horizontalModifier, j + verticalModifier, piecesCopy).Any())
        {
            possibleMoves.Add(new CoordinatePair(i, j, i + horizontalModifier,
                j + verticalModifier));
        }
    }

    private void SwitchWithNeighbor(ArrayWrapper<Piece> piecesCopy, int j, int i,
        int horizontalModifier, int verticalModifier)
    {
        Piece tempPiece = piecesCopy[i + horizontalModifier, j + verticalModifier];

        if (tempPiece != null)
        {
            piecesCopy.SetElement(i + horizontalModifier, j + verticalModifier, piecesCopy[i, j]);
            piecesCopy.SetElement(i, j, tempPiece);

            piecesCopy[i, j].SetCoordinates(i, j);
            piecesCopy[i + horizontalModifier, j + verticalModifier].SetCoordinates(
                i + horizontalModifier, j + verticalModifier);
        }
    }
}

{% endhighlight %}

Lots of other code got new comments and went through refactoring to improve readability and efficiency.
