# Train Tale

## *3D Project - at Yrgo Game Creator*
[Itch Page](https://yrgo-game-creator.itch.io/train-tale)\
[Trailer](https://www.youtube.com/watch?v=okvqh6uOwDE)

During my second semester at Yrgo (Game Creator Programmer) I had the pleasure to work with a wonderful team of fellow students in creating this stunning piece of game.\
 \
*Developed in Spring/Summer of 2025*
_____________________________________________________________________________________
Working on this project, one of my personal goals was to get more comfortable with taking a step back and make sure I could read and understand my own code. One of my teachers gave me a valuable tip that I will take with me wherever I go - bring out the old trusty pen and paper and write down the steps I needed to take to get where I wanted. Looking back I can tap myself on the shoulder and say that I reached that goal, and I learned so much more in the process.

## A Selection of Contributions
### Events
1. Following eyes\
![](https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Following_Eyes.gif)

<details>
<summary>Show Following_Eyes.cs</summary>
<br>

```ruby
{

    public GameObject[] eyeBalls;

    [SerializeField]
    private GameObject player;

    [SerializeField] private float maxDistance = 5f;
    [SerializeField] private float minDistance = -5f;
    private float rotationTime = 1f;
    private Vector3 playerPos;
    private Quaternion startRotation;

    private void Start()
    {
        startRotation = transform.rotation;
    }

    void Update()
    {
        if (!FindAnyObjectByType<CheckForObstacle>().IsPlayerHiding())
        {
            CalculateRotation();
        }
    }

    private void CalculateRotation()
    {
        foreach (GameObject eyeball in eyeBalls)
        {
            playerPos = player.transform.position;
            Vector3 distance = transform.position - playerPos;
            Quaternion resetRotation = Quaternion.Euler(0, 0, 0);
            Quaternion lookRotation = Quaternion.LookRotation((playerPos - eyeball.transform.position).normalized);

            if (distance.x < maxDistance && distance.x! > minDistance ||
                distance.x > minDistance && distance.x! < maxDistance)
            {
                eyeball.transform.rotation = Quaternion.Slerp(eyeball.transform.rotation,
                lookRotation, rotationTime * Time.deltaTime);
            }

            else
            {
                eyeball.transform.rotation = Quaternion.Slerp(eyeball.transform.rotation,
                resetRotation, rotationTime * 0.5f * Time.deltaTime);
            }
        }
    }

    public void GetStartRotation()
    {
        eyeBalls[0].transform.rotation = Quaternion.Slerp(transform.rotation,
                startRotation, rotationTime * Time.deltaTime);
    }
}
```
</details>


### *Developed by*
![](https://github.com/ewigur/Portfolio/blob/main/Train%20Tale/GIFs/Carneval.gif)

It was challenging, but most of all it was fun - and I really hit the jackpot when I signed up to work with this team of aspiring game devs!
