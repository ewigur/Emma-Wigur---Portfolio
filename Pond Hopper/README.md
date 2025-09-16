<a name="TOP"></a>

<p align="center">
  <img src=https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH.gif />
</p>


# Brief summary
*Pond Hopper* was created for an assignment at *Yrgo Game Creator Programmer*.
 \
 \
The purpose was to build a game for android, and to use a selection of programming patterns such as a **state machine**, **object pooling** and **scriptable objects**.
 \
 \
The result was this little gem, and being the first game I finished on my own it definitely has a place on the show off -list.

## 
[Itch Page](https://ewigur.itch.io/pond-hopper)
## 

 2D game for mobile\
*January 2025 - February 2025*
_____________________________________________________________________________________

## Mechanics

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_GamePlay.gif)

**1.1 Jump**\
The core mechanic of this game is pretty straight forward - touch, drag and release to make the frog jump and devour the flies.
The indicator shows direction of the jump, which gives the player control over how far the frog will go.
The double jump was the best way to balance the mechanic, since this little critter can't swim a miscalculated jump == drowning.
 \
 \
 **1.2 Pickups**\
Each collected fly adds to the score. The base of the fly is built on a scriptable object, which was the best approach for managing the attributes.\
I decided on having one common fly, and one firefly that would be more rare but with a higher score on collection. I quickly noticed that the firefly
caught the eye of the people testing the game,and it became more interesting gameplay as the subjects were more likely to chase the shiny fly instead of
diving into the cloud of common flies that might even give them a higher score.

*[Code can be found further down the page]*
_____________________________________________________________________________________

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_ObjectPool.gif)

**2. Object Pool**

Since were already on the topic of the flies - they also have their very own object pool. Since the gameloop goes on and on, 
it would be irresposible of me not to implement a _circle of life_ kind of functionality. The flies spawn from a pool of preloaded
prefabs, and when the player collects them they return to the pool to be released again. 

*[Code can be found further down the page]*
_____________________________________________________________________________________

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_HS.gif)

**3. Highscore & Leaderboard**

Another system I wanted to implement was a leaderboard. I decided to only make it local, since this game was more about making it for myself and a fun thing to show friends and family (and, of course, you). 
If the player reaches a score higher than the last 8, they will be prompted to add their name in the textbox upon the frogs final death. The highscore is saved on the local device, and the leaderboard will be updated and available in the main menu of the game.

_____________________________________________________________________________________

## Graphics

All graphics are created by me.\
The only exception is the level background,\
which is an AI-generated image (Adobe Firefly) that I repainted and cut into three different pieces to layer the game scene.

| Fly  | Firefly |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/Fly.gif)  | ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/FireFly.gif) |

| Level  | Frog |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/Level.gif)  |  ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/PH_Frog.gif) |

| Platforms | 
| ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/Graphics/PH_Log_Stone.png) |

_____________________________________________________________________________________
## Code blocks, *for the curious*

<details>
<summary>PickUpItem.cs - Scriptable Object</summary>
<br>
  
```ruby
/*NOTE: This is the item data container for the pickups (flies).
In addition to defining what kind of item this is, this is also used
by the object pool to calculate which of the two pickup items to choose  - based on spawnProbability*/

[CreateAssetMenu(fileName = "PickUp", menuName = "ScriptableObjects/PickUp Item", order = 1)]
public class PickUpItem : ScriptableObject
{
    public string itemName;
    
    public Animator pickUpAnimator;
    public float flockMovement;
    public GameObject prefab;
    public int spawnAmount;
    public int value;
    
    [Range(0f, 1f)]
    public float spawnProbability;
}

```

</details>

<details>
<summary>PickUpPool.cs - Object Pool</summary>
<br>
  
```ruby

/*
  NOTE: A snippet from the object pool.
        I used Unity's built in OP, and it takes information from the behavioural script created for the flies,
        which in turn is based off of the scriptable object that contains all the data.
*/


/*
  NOTE: The pool takes the "spawnProbability" (from the scriptable object) into account,
        and releases a set amount of flies based on weight and amount of flies already excisting in the scene.
*/
________________________

/*
  Snippet 1  - Getting item data
*/

    private PickUpItem GetRandomPickUpItem()
    {
        float totalWeight = pickUpItems.Sum(item => item.spawnProbability);
        float randomValue = Random.Range(0f, totalWeight);
        float cumulativeWeight = 0f;

        foreach (var item in pickUpItems)
        {
            cumulativeWeight += item.spawnProbability;
            if (randomValue <= cumulativeWeight)
                return item;
        }

        return pickUpItems[0];
    }
________________________
/*
  Snippet 2  - Spawn Method
*/

    private void Spawn()
    {
        if (currentActivePickUps >= maxActivePickUps) 
            return;

        var randomPickUpItem = GetRandomPickUpItem();

        for (var i = 0; i < randomPickUpItem.spawnAmount; i++)
        {
            var pickUp = pickUpPools[randomPickUpItem].Get();
            currentActivePickUps++;

            pickUp.transform.position = GetRandomSpawnPosition();
            pickUp.Initialize(randomPickUpItem);
            pickUp.OnReturn += DisablePrefab;
        }
    }
________________________
/*
  Snippet 3  - Return item to pool
*/

    private void DisablePrefab(PickUpBehaviour pickUp)
    {
        if (pickUpPools.TryGetValue(pickUp.GetItemData(), out var pool))
        {
            currentActivePickUps--;
            pool.Release(pickUp);
            pickUp.OnReturn -= DisablePrefab;
        }
    }
}

```

</details>

<details>
<summary>State Machine</summary>
<br>
  
```ruby
/*
  NOTE: This is a snippet of what happens under the hood as the game changes states.
        I created enums for each state
*/
      public enum GameStates
    {
        MainMenu,
        GameLoop,
        GamePaused,
        GameResumed,
        GameRestarted,
        GameOver,
    }

________________________

*/
    NOTE: As soon as the game state changes, the corresponding components listens to that.
          Below is a snippet from under the hood upon player death...
*/

    private void HandleStates(GameStates newState)
    {
        switch (newState)
        {
            
            case GameStates.GameOver:
                onToggleInput?.Invoke(false);
                TriggerPauseMusic?.Invoke();
                Time.timeScale = 0f;
                break;
        }
    }

________________________

/*
    NOTE: ...and a bunch of happens in correlation with the state change.
             (UI managingin, this case.)
*/

(from "InGameStatesHandler")

{
    private void GameOver()
    {
        GMInstance.ChangeState(GameStates.GameOver);
        livesDisplay.SetActive(false);
        pauseButton.SetActive(false);
        gameOverMenu.SetActive(true);
    }
}

```

</details>
_____________________________________________________________________________________

### *Developed by*
Emma Wigur
_____________________________________________________________________________________
### *Made Possible by*
![Image](https://github.com/ewigur/Portfolio/blob/main/ThumbNails/Yrgo.png)
*Higher Vocational Education - Game Creator Programmer, Göteborg*
_____________________________________________________________________________________

[RETURN TO TOP](#TOP)
             <a name="TOP"></a>  
