---
title:  "Match 3 game Part 4 - Color bomb, score calculation, GUI basics"
date:   2017-07-25 20:00:00 +0200
tags: match3 GUI
---
A color bomb piece has been added to the game as well as score calculation and GUI elements.
<!--more-->

I wanted to add another piece popular in match 3 games - a color bomb which removes all the pieces of a certain color when switched with a regular piece (or removes all pieces if switched with another color bomb). This bomb is created by matching 5 or more pieces of the same color.

A new Color value was added to the **BombType** enum. **Piece** class also got a new method **HasColorBomb()** making it easy to find out if a piece has a color bomb attached. **PieceManager** has been expanded with a **GetColorBombMatches()** method which returns all the matches involved in a color bomb switch. To make it work, another method called **GetPiecesOfType()** was implemented which returns all the pieces of a certain type.

Integ tests and unit tests were also added to follow the changes introduced by color bomb logic. A GUI object was created to accommodate a simple score counter. To achieve a working score counter I added a few more classes: **GameManager**, **GuiManager** and the **CSharpMessenger** class already used in my FPS base project to make communication between components more efficient.

**GameManager** handles current score and a score limit (so the game can be won by reaching a target score). It listens to the **AddScore** event and also listens to **OnBoardInitialized** to avoid problems caused by score calculations during board initalization. **GameManager** creates the **GameManager** instance which has the same GameObject as parent.

**AddScore** event is fired in **PieceManager**'s **AddScore()** method which gets the score value from the removed piece, adds a bomb score value if it has a bomb attached and also multiplies it with a cascade multiplier. Multiplier is incremented after every consecutive match created in a single move.

If a score limit is set on the current level, a **GameOverWin** event is fired by **GameManager** upon reaching this target score along with a message to display on the GUI.

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using UnityEngine;

public class GameManager : MonoBehaviour
{
    [SerializeField]
    private int score = 0;
    [SerializeField]
    private int scoreLimit = 0;
    [SerializeField]
    private GuiManager guiManager;

    private bool isBoardInitialized = false;

    private void Start()
    {
        guiManager = gameObject.AddComponent<GuiManager>();
        guiManager.Init();
    }

    public void ResetScore()
    {
        score = 0;
        guiManager.SetScore(score);
    }

    private void OnEnable()
    {
        Messenger<int>.AddListener(GameEvent.AddScore, OnAddScore);
        Messenger.AddListener(GameEvent.BoardInitialized, OnBoardInitialized);
    }

    private void OnDisable()
    {
        Messenger<int>.RemoveListener(GameEvent.AddScore, OnAddScore);
        Messenger.RemoveListener(GameEvent.BoardInitialized, OnBoardInitialized);
    }

    private void OnAddScore(int value)
    {
        if (isBoardInitialized)
        {
            score += value;
            guiManager.SetScore(score);

            if(scoreLimit != 0 && score >= scoreLimit)
            {
                Messenger<string>.Broadcast(GameEvent.GameOverWin,
                    "You have reached the target score!");
            }
        }
    }

    private void OnBoardInitialized()
    {
        isBoardInitialized = true;
    } 
}

{% endhighlight %}

**GuiManager** puts all the GUI texts attached to its GameObject in a List, this way all text elements can be managed as a whole. It has a text for the score and two texts for the game over canvas group which is set to alpha 0 by default. **SetScore()** can be used to set the score text element's value. Upon receiving an **OnGameOverWin** message it displays the game over canvas group along with the appropriate message.

![Obstacle tiles]({{site.url}}/assets/images/you-win-gui.PNG){: .align-center}
*Target score reached*
{: .text-center}

[Copy code snippet](#link){: .btn}  

{% highlight c# %}

using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class GuiManager : MonoBehaviour
{

    [SerializeField]
    private GameObject gui;
    private Text scoreText;
    private Text bigGameOverText;
    private Text smallGameOverText;
    private CanvasGroup gameOverGroup;

    public void Init()
    {
        gui = GameObject.Find("Gui");
        List<Text> guiTexts = new List<Text>();
        gui.GetComponentsInChildren(guiTexts);
        scoreText = guiTexts.Find(t => t.name == "ScoreText");        
        SetScore(0);

        gameOverGroup = gui.GetComponentInChildren<CanvasGroup>();
        bigGameOverText = guiTexts.Find(t => t.name == "BigText");
        smallGameOverText = guiTexts.Find(t => t.name == "SmallText");
    }

    public void SetScore(int score)
    {
        scoreText.text = score.ToString();
    }

    private void OnEnable()
    {
        Messenger<string>.AddListener(GameEvent.GameOverWin, OnGameOverWin);
        
    }

    private void OnDisable()
    {
        Messenger<string>.RemoveListener(GameEvent.GameOverWin, OnGameOverWin);        
    }

    private void OnGameOverWin(string smallText)
    {
        bigGameOverText.text = "You Win";
        smallGameOverText.text = smallText;
        gameOverGroup.alpha = 1f;
    }
}

{% endhighlight %}