<a name="TOP"></a>

![PogoLogo](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/Img/pogologo.png)


# Game Overview
Pogo Pirates is all about being the King of the hill. Eliminate your friends (or enemies) with your mighty pogo stick; the perfect party game for couch hangouts!

## 
[Itch Page](https://yrgo-game-creator.itch.io/pogopirates)\
[Trailer](https://www.youtube.com/watch?v=AxzvmTWsCbA)
## 

 \
*8 week 2D Project - at Yrgo Game Creator*\
*Developed November 2024 - January 2025*
_____________________________________________________________________________________

*This project was the starting point of getting comfortable with building games, as well as working in a*\
*team. The latter wasn't hard, since every single one in the team really kept on encouraging and supporting eachother,*\
*which also made the first point not as intimidating.*\
 \
*This was the first game I had ever been working on, and I couldn't be more proud of how it turned out*

## A Selection of Contributions

**1. Camera**
 \
The camera system is based on fast paced action PvP games like *"Super Smash Bros"*. The camera keeps track of how many players are currently in the game, and adjusts a bounding box accordingly. The camera itself moves only on the X-axis, and zooms to get the feeling of depth. A dynamic camera was important to get the right feel of the game, and it was a good experience to keep that as my main focus.

![Camera](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/CamShow.gif)

</details>

<details>
<summary>CameraManager.cs</summary>
<br>
  
```ruby
namespace PogoPirates.Cameras
{
    public class CameraManager : MonoBehaviour
    {
        [Header("Contents")]
        private GameObject[] targets = new GameObject[4];

        [SerializeField] private GameObject[] arrows;

        [Header("Arrow tweaks")]
        [Range(0.001f, 0.95f)]
        [SerializeField] private float arrowoffset1 = 0.05f;

        [Range(0.001f, 0.95f)]
        [SerializeField] private float arrowoffset2 = 0.95f;

        [Header("Attributes")]
        [SerializeField]private float minValue = 7;
        [SerializeField]private float maxValue = 10;

        [Range(0.0001f, 0.005f)]
        [SerializeField] private float offsetMultiplier = 0.001f;

        [Range(0, 1)]
        [SerializeField] private float zoomSmoother = 0.001f;
        private Camera cam;
        private Vector3 boundsCenter;
        private Bounds playerBounds;

        [SerializeField] private float cameraExpand = 7f;
        
        public void Start()
        {
            cam = GetComponent<Camera>();

            foreach (GameObject playerObject in InstanceManager.Instance.playerObjects)
            {
                if (playerObject is null)
                    continue;
                
                Player player = playerObject.GetComponent<Player>();
                targets[player.id] = playerObject;
            }

            for (int i = 0; i < targets.Length; i++)
            {
                Transform arrowTransform = transform.Find($"Arrow_{i}");
                
                if (arrowTransform != null)
                {
                    arrows[i] = arrowTransform.gameObject;
                    arrows[i].GetComponent<SpriteRenderer>().enabled = false;
                }
            }

            FollowPlayers();           
        }

        private void OnEnable()
        {
            Player.onJoin += PlayerJoins;
            Player.onDeath += OnPlayerDeath;
        }
        
        private void FollowPlayers()
        {   
            if (targets.All(item => item is null))
            {
                playerBounds = new Bounds(Vector3.zero, Vector3.zero);
            }
            else if (targets.Count(item => item is not null) == 1)
            {
                GameObject target = targets.First(item => item is not null);
                
                playerBounds = new Bounds(target.transform.position, Vector3.zero);
            }
            else
            {
                GameObject target = targets.Where(item => item is not null).Skip(1).First();

                playerBounds = new Bounds(target.transform.position, Vector3.zero);
            }

            for (int i = 0; i < targets.Length; i++)
            {
                if (targets[i] is null)
                    continue;
                
                playerBounds.Encapsulate(targets[i].transform.position);
                ArrowIndicator(targets[i].GetComponent<Player>());
            }
            
            playerBounds.Expand(cameraExpand);
            
            boundsCenter.x = Mathf.Clamp(boundsCenter.x, -cam.orthographicSize * cam.aspect, cam.orthographicSize * cam.aspect);
            boundsCenter.y = Mathf.Clamp(boundsCenter.y, -cam.orthographicSize, cam.orthographicSize);

            Vector3 moveOffset = playerBounds.center - transform.localPosition;
            moveOffset.z = 0;

            if(moveOffset.sqrMagnitude > 4)
            {
                transform.localPosition += moveOffset * (offsetMultiplier * 20);
            }
            else if(moveOffset.sqrMagnitude > 2)
            {
                transform.localPosition += moveOffset * (offsetMultiplier * 10);
            }
            else if(moveOffset.sqrMagnitude > 0.1f)
            {
                transform.localPosition += moveOffset * offsetMultiplier;
            }

            if (playerBounds.size.x > playerBounds.size.y)
            {
                float orthSize = Mathf.Lerp(Mathf.Clamp(playerBounds.extents.x / cam.aspect, minValue, maxValue), cam.orthographicSize, zoomSmoother);
                
                cam.orthographicSize = orthSize;
            }
            else
            {
                float orthSize = Mathf.Lerp(Mathf.Clamp(playerBounds.extents.y, minValue, maxValue), cam.orthographicSize, zoomSmoother);
                cam.orthographicSize = orthSize;
            }              
        }

        private void ArrowIndicator(Player player)
        {
            Vector3 viewportPos = cam.WorldToViewportPoint(player.transform.position);

            GameObject arrow = arrows[player.id];

            bool isOutsideView = viewportPos.x < 0 || viewportPos.x > 1 || viewportPos.y < 0 || viewportPos.y > 1;
            arrows[player.id].SetActive(isOutsideView);
            
            if (isOutsideView)
                arrow.GetComponent<SpriteRenderer>().enabled = true;
            else
                arrow.GetComponent<SpriteRenderer>().enabled = false;

            if (isOutsideView)
            {               
                Vector3 arrowPosition = viewportPos;
                arrowPosition.x = Mathf.Clamp(arrowPosition.x, arrowoffset1, arrowoffset2);
                arrowPosition.y = Mathf.Clamp(arrowPosition.y, arrowoffset1, arrowoffset2);

                Vector3 worldPosition = cam.ViewportToWorldPoint(new Vector3(arrowPosition.x, arrowPosition.y, cam.nearClipPlane));
                arrow.transform.position = worldPosition;

                Vector3 direction = player.transform.position - arrow.transform.position;
                float angle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;
                arrow.transform.rotation = Quaternion.Euler(0, 0, angle);
            }
        }

        public void FixedUpdate()
        {
            FollowPlayers();
        }

        public void PlayerJoins(Player player)
        {
            arrows[player.id].GetComponent<SpriteRenderer>().enabled = true;
            targets[player.id] = player.gameObject;
        }

        private void OnPlayerDeath(int id)
        {           
            arrows[id].GetComponent<SpriteRenderer>().enabled = false;
            targets[id] = null;
        }

        private void OnDisable()
        {
            Player.onJoin -= PlayerJoins;
            Player.onDeath -= OnPlayerDeath;
        }
    }

```
</details>
_____________________________________________________________________________________

**2. Angled Platforms**
 \
To keep things interesting the platforms at the far end of the ship on this level has an angled bounce effect.\
Landing on them gives the player an angled push, that can be adjusted to line up with the angle of the platform.

![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/AngledPlatforms.gif)

</details>

<details>
<summary>AngledPlatform.cs</summary>
<br>
  
```ruby

    public class AngledPlatform : MonoBehaviour
    {
        [Tooltip("The angle of the platform (game object)")]
        [Range(-180, 180)]
        [SerializeField] 
        private int angle = 45;

        [Tooltip("The force of which the player is pushed away on contact")]
        [Range(15f, 50f)]
        [SerializeField]
        private float forceStrength = 30f;
        public void OnCollisionEnter2D(Collision2D collider) 
        {
            
            if (!collider.gameObject.TryGetComponent(out Player player))
                return;

            Rigidbody2D playerRigidbody = player.GetComponent<Rigidbody2D>();
            if (playerRigidbody == null)
                return;

            Quaternion platformRotation = Quaternion.Euler(0, 0, angle);
            player.transform.rotation = platformRotation;

            Vector2 forceDirection = transform.up;
            playerRigidbody.AddForce(forceDirection * forceStrength, ForceMode2D.Impulse);
        }
    }

```
</details>
_____________________________________________________________________________________

**3. In game UI**
 \
Players don't have a health bar, instead the hit percentage goes up as players hit one another.\
I worked with the hit progression UI, and added:
- Color lerp and animation on numbers when numbers change
- Plank shake animation

An arrow shows up to indicate where a player is when they go outside of the camera bounds.\
My work on this was:
- Create an indicator based on the characters main color
- Implement a system that tracks if the player is outside of camera bounds that would trigger the indicator

| Hit Percent  | Indicator |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/UIShake.gif)  | ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/Indicator.gif) |
_____________________________________________________________________________________
**4.1 UI**
 \
The main animations for the menu and tutorial is made by one of the artists on the project, my contributions were:

**4.2 Animation**
- Button shakes in menues
- Show/Hide tutorial side panels

 \
**4.3 Programming**
- Menu behaviours
- Tutorial panel behaviours

| Menu  | Tutorial |
| ------------- | ------------- |
| ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/UI_1.gif)  | ![](https://github.com/ewigur/Portfolio/blob/main/Pogo%20Pirates/GIFs/UI_2.gif) |
_____________________________________________________________________________________

### *Game Created At*
![Image](https://github.com/ewigur/Portfolio/blob/main/ThumbNails/Yrgo.png)
*Higher Vocational Education - Game Creator Programmer, Göteborg*
_____________________________________________________________________________________

[RETURN TO TOP](#TOP)
             <a name="TOP"></a>  
