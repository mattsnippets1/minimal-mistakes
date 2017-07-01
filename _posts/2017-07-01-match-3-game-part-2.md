---
title:  "Match 3 game Part 2 - Obstacles and bombs"
date:   2017-07-01 10:00:00 +0200
tags: Match3
---
New elements have been added to the game: bomb pieces and indestructible obstacle tiles.
<!--more-->

While the column slide effect was already included in the previous version it would only work during piece removal and refilled pieces simply popped out of nowhere. This was quite easy to fix: the **CreatePiece()** method now has a vertical offset and a movement time parameter. If the offset is bigger than 0, the pieces are instantiated above the board and "fall into place" with the given speed. Notice that **CreateRandomPiece()** has been rewritten into the more generic **CreatePiece()** method so it is now possible to create a certain kind of gamepiece.

Another new feature is the introduction of obstacle tiles. The player cannot interact with these and they block gamepieces. They can be used to create a more interesting level shape. The **Tile** class got a brand new **TileType** enum which currently contains Basic and Obstacle. Gamepiece fill and column slide now only work if the tile is not an obstacle. **Board** also got an array containing **PresetTile** structures so it is much easier to set up a starting board state from the editor.

![Obstacle tiles]({{site.url}}/assets/images/obstacle-tiles.PNG){: .align-center}
*Obstacle tiles*
{: .text-center}

The biggest addition since the last version is the inclusion of bombs. I considered different approaches and finally decided to implement bombs as separate GameObjects attached to gamepieces. They have their own sprite renderers and **Bomb** scripts. This script doesn't contain much: the type of the bomb, an **Init()** method and some code to automatically replace the bomb sprite if the bombtype is altered in the editor. The sprites are only simple markers which appear on a layer above the gamepieces. This way I can see immediately that the gamepiece has a certain kind of attached bomb.

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

public class Bomb : MonoBehaviour
{
    [SerializeField]
    private BombType bombType;
    [SerializeField]
    private Sprite[] sprites;

    private SpriteRenderer spriteRenderer;

    public void Init(BombType type)
    {
        bombType = type;
    }

    public BombType BombType
    {
        get
        {
            return bombType;
        }
    }

    private void Start()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void OnValidate()
    {
        if (spriteRenderer != null)
        {
            spriteRenderer.sprite = sprites[(int)bombType];
        }
    }
}

public enum BombType
{
    Block,
    Row,
    Column
}

{% endhighlight %}

To make the bombs actually remove gamepieces the **Board** class has been extended with some extra code. In **ClearCoroutine()** a new method called **ExplodeBombs()** is invoked. This finds all the bombs in the current matching pieces then calls **GetExplodedPieces()** which returns with a collection of pieces destroyed by the given bomb type (to get the pieces, three new methods are used from **PieceManager**, namely **GetPieceBlock()**, **GetPieceColumn()** and **GetPieceRow()**). The bombs trigger each other so a chain reaction can occur between bombs. That's why **ExplodeBombs()** is recursively called until no more explosions can occur within the effected area. The collection of exploded pieces is then included in the collection of pieces to be removed. Here is the code of **ExplodeBombs()** and **GetExplodedPieces()**.

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

private void ExplodeBombs(IEnumerable<Piece> pieces, HashSet<Piece> allPieces)
{
    HashSet<Piece> explodedPieces = new HashSet<Piece>();

    IEnumerable<Piece> piecesWithBomb = pieces.Where(piece => piece != null &&
    piece.GetComponentInChildren<Bomb>() != null);

    foreach (Piece bPiece in piecesWithBomb)
    {
        explodedPieces.UnionWith(GetExplodedPieces(bPiece));
    }

    IEnumerable<Piece> effectedPieces = explodedPieces.Except(piecesWithBomb);

    if (explodedPieces.Except(allPieces).Any())
    {
        allPieces.UnionWith(explodedPieces);
        ExplodeBombs(effectedPieces, allPieces);
    }
}

private IEnumerable<Piece> GetExplodedPieces(Piece bombPiece)
{
    IEnumerable<Piece> explodedPieces;

    switch (bombPiece.GetComponentInChildren<Bomb>().BombType)
    {
        case BombType.Block:
            explodedPieces = pieceManager.GetPieceBlock(bombPiece.X, bombPiece.Y, 1);
            break;
        case BombType.Row:
            explodedPieces = pieceManager.GetPieceRow(bombPiece.Y);
            break;
        case BombType.Column:
            explodedPieces = pieceManager.GetPieceColumn(bombPiece.X);
            break;
        default:
            Debug.LogError("Unknown bomb type!");
            explodedPieces = null;
            break;
    }

    return explodedPieces;
}

{% endhighlight %}


<iframe width="560" height="315" src="https://www.youtube.com/embed/8l2JZm_exUQ?rel=0" frameborder="0" allowfullscreen></iframe><br>