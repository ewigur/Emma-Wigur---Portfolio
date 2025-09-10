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
_____________________________________________________________________________________

**2. Object Pool**

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_ObjectPool.gif)

_____________________________________________________________________________________

**3. SomeText - Highscore**

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/PH_HS.gif)

_____________________________________________________________________________________

**4. SomeText - Game Adjustments**

![](https://github.com/ewigur/Portfolio/blob/main/Pond%20Hopper/GIFs/Sliders.gif)

_____________________________________________________________________________________

**4. Graphics**

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
