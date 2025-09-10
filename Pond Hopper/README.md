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

**2. Object Pool**

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_ObjectPool.gif)

Since were already on the topic of the flies - they also have their very own object pool. Since the gameloop goes on and on, 
it would be irresposible of me not to implement a _circle of life_ kind of functionality. The flies spawn from a pool of preloaded
prefabs, and when the player collects them they return to the pool to be released again. 

*[Code can be found further down the page]*
_____________________________________________________________________________________

**3. Highscore & Leaderboard**

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_HS.gif)

Another system I wanted to implement was a leaderboard. I decided to only make it local, since this game was more about making it for myself and a fun thing to show friends and family (and, of course, you). 
If the player reaches a score higher than the last 8, they will be prompted to add their name in the textbox upon the frogs final death. The highscore is saved on the local device, and the leaderboard will update and available in the main menu of the game.

_____________________________________________________________________________________

**4. Audio Adjustments**

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/Sliders.gif)

I was really proud of this nifty little menu feature. I spent a lot of time finding the right music/SFX for this game, and I really wanted to create a system to manage sound so every player could tailor audio based on their own preferences. 

_____________________________________________________________________________________

**5. Graphics**

_____________________________________________________________________________________
**6. Code blocks, *for the curious***

<details>
<summary>PickUpItem.cs - Scriptable Object</summary>
<br>
  
```ruby
using UnityEngine;

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
using Random = UnityEngine.Random;
using System.Collections.Generic;
using UnityEngine.Pool;
using UnityEngine;
using System.Linq;

public class PickUpPool : MonoBehaviour
{
    [SerializeField] private List<PickUpItem> pickUpItems;
    [SerializeField] private int defaultCapacity = 5;
    [SerializeField] private int maxActivePickUps = 10;
    [SerializeField] private float timeBetweenSpawns = 0.2f;
    [SerializeField] private float startSpawnTime = 0.2f;

    [SerializeField] private float minX = -7f;
    [SerializeField] private float maxX = 7f;
    [SerializeField] private float minY = 2f;
    [SerializeField] private float maxY = 4f;

    private int currentActivePickUps;
    
    private Dictionary<PickUpItem, ObjectPool<PickUpBehaviour>> pickUpPools;

    private void Start()
    {
        pickUpPools = new Dictionary<PickUpItem, ObjectPool<PickUpBehaviour>>();
        InitializePickUpPools();
        InvokeRepeating(nameof(Spawn), startSpawnTime, timeBetweenSpawns);
    }

    private void InitializePickUpPools()
    {
        foreach (var item in pickUpItems)
        {
            pickUpPools[item] = CreatePoolForItem(item);
            
            for (int i = 0; i < defaultCapacity; i++)
            {
                var pickUp = pickUpPools[item].Get();
                pickUpPools[item].Release(pickUp);
            }
        }
    }

    private ObjectPool<PickUpBehaviour> CreatePoolForItem(PickUpItem item)
    {
        return new ObjectPool<PickUpBehaviour>
        (
            createFunc: () =>
            {
                var instance = Instantiate(item.prefab).GetComponent<PickUpBehaviour>();
                instance.gameObject.SetActive(false);
                return instance;
            },
            actionOnGet: pickUp => pickUp.gameObject.SetActive(true),
            actionOnRelease: pickUp => pickUp.gameObject.SetActive(false),
            actionOnDestroy: pickUp => Destroy(pickUp.gameObject),
            collectionCheck: false, defaultCapacity, maxActivePickUps
        );
    }

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
    
    private Vector2 GetRandomSpawnPosition()
    {
        float randomX = Random.Range(minX, maxX);
        float randomY = Random.Range(minY, maxY);
        Vector2 spawnPosition = new Vector2(randomX, randomY);
        
        return spawnPosition;
    }

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
<summary>GameManager.cs - Game State Machine</summary>
<br>
  
```ruby
using System;
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager GMInstance;
    
    public static Action<GameStates> onGameStateChanged;
    public static Action<bool> onToggleInput;
    public static Action TriggerMenuMusic;
    public static Action TriggerGameMusic;
    public static Action TriggerPauseMusic;
    public static Action TriggerResumeMusic;
    
    
    public enum GameStates
    {
        MainMenu,
        GameLoop,
        GamePaused,
        GameResumed,
        GameRestarted,
        GameOver,
    }

    public GameStates state;
    private void Awake()
    {
        if (GMInstance != null)
        {
            Destroy(gameObject);
        }

        else
        {
            GMInstance = this;
            DontDestroyOnLoad(gameObject);
            ChangeState(GameStates.MainMenu);
        }
    }

    public void ChangeState(GameStates newState)
    {
        if(state == newState)
            return;

        state = newState;
        onGameStateChanged?.Invoke(state);
        HandleStates(newState);
    }

    private void HandleStates(GameStates newState)
    {
        switch (newState)
        {
            case GameStates.MainMenu:
                TriggerMenuMusic?.Invoke();
                Time.timeScale = 1;
                break;
            
            case GameStates.GameLoop:
                onToggleInput?.Invoke(true);
                PlayerPrefs.SetInt("currentScore", 0);
                TriggerGameMusic?.Invoke();
                Time.timeScale = 1f;
                break;
            
            case GameStates.GamePaused:
                onToggleInput?.Invoke(false);
                TriggerPauseMusic?.Invoke();
                Time.timeScale = 0f;
                break;
            
            case GameStates.GameResumed:
                onToggleInput?.Invoke(true);
                TriggerResumeMusic?.Invoke();
                Time.timeScale = 1f;
                break;
            
            case GameStates.GameRestarted:
                onToggleInput?.Invoke(true);
                TriggerResumeMusic?.Invoke();
                Time.timeScale = 1f;
                break;
            
            case GameStates.GameOver:
                onToggleInput?.Invoke(false);
                TriggerPauseMusic?.Invoke();
                Time.timeScale = 0f;
                break;
        }
    }
    
    public void OnApplicationQuit()
    {
        #if UNITY_EDITOR
                UnityEditor.EditorApplication.isPlaying = false;
        #endif
        
        PlayerPrefs.DeleteKey("SFXVolume");
        PlayerPrefs.DeleteKey("ButtonsVolume");
        PlayerPrefs.DeleteKey("MusicVolume");
        PlayerPrefs.DeleteKey("remainingLives");
        PlayerPrefs.DeleteKey("currentScore");
        
        Debug.Log("Keys restored");
        
        Application.Quit();
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
