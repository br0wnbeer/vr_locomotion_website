+++
title = 'Implementation Locomotion'
date = 2024-03-30T21:49:56+01:00
draft = false
summary = "Implementation of the locomotion mechanic"
+++
# Introduction
When starting the implementation process I first looked if someone else had implemented a similar thing in Unity before. As far as my research concerns, there was no prior implementation of a locomotion method similar to mine. Anyway for the first step of my implementation ,I found some tutorials implementing projectile system. By default this also became my first step then too.

## Projectile System 
For the implementation of the projectile system I fist created a **Prefab** in Unity for my bullet. 


{{<img1 src = "../../post/rocket.png" caption = "Rocket Prefab">}}

During development I chose this as a model for the bullet because it was easy to spot and just a simple way to represent of a rocket. This then later also stayed like that do to my blender skills just not being the best.  How the rocket itself behaves is given in the following code. 

{{< highlight csharp "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
    public class rocket_behaviour : MonoBehaviour
{
    [SerializeField] private float speed = 20.0f;
    [SerializeField]public float strength = 1000;
    [SerializeField] float radius = 20.0f;
    [SerializeField] public GameObject explosion;

    [SerializeField] private AudioClip clip;
    // Start is called before the first frame update
    void Start()
    {
        // Init speed and rotation of the bullet 
        GetComponent<Rigidbody>().AddForce(transform.forward * speed, ForceMode.Impulse);
        transform.rotation = Quaternion.Euler(90,0,0);
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    private void OnCollisionEnter(Collision other)
    { 
        // Collision with other object in the scene 
        Collider[] colliders = Physics.OverlapSphere(transform.position, 20f);
        foreach (Collider c in colliders)
        {
            Rigidbody rigidbody = c.GetComponent<Rigidbody>();
            if (rigidbody != null)
            {
                // Explosion animation , force and sound added
                rigidbody.AddExplosionForce(strength, transform.position, radius, 3.0f);
                Destroy(this.gameObject);
                var hit = Instantiate(explosion, transform.position, transform.rotation);
                AudioSource.PlayClipAtPoint(clip,transform.position);
                Destroy(hit , 2);
                
            }
        }
    }
}
{{< / highlight >}}

Here you can set the speed the rocket is moving as well as the strength and radius of the explosion given by the Serialized fields. I also included two more field for an explosion sound as well as a explosion animation. For the explosion animation I used the [Particle Pack](https://assetstore.unity.com/packages/vfx/particles/particle-pack-127325) from the Unity store a well as an free explosion sound from [upbeat.io](https://uppbeat.io/browse/sfx/explosions). I tried to write my code in a way that it is quite self explanatory, this is the reason for not further going into an explanation here.
### Problems 
Some problems I encountered with the projectile system were to find a explosion force value that did not shoot the player up to high and is not so low that the player does not move at all. There also was allot of reading Unity documentation involved about functionalities that Unity already provides regarding explosions. This resulted in allot of time being lost to finding the right Unity functionality.

## OVR Camera Rig Locomotion
After creating the Rocket as well its behavior in the scene my next step was to implement the locomotion itself. For this I turned on the Gravity on the OVR Camera Rig in the Scene and set the isTrigger value of the Rigidbody to false. 

Then I started editing the Locomotion Technique class given to us by the professor and stated editing it to suit my needs.
{{< highlight csharp "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
public class LocomotionTechnique : MonoBehaviour
{
    // Please implement your locomotion technique in this script. 
    public OVRInput.Controller leftController;
    public OVRInput.Controller rightController;
    [Range(0, 10)] public float translationGain = 0.5f;
    public GameObject hmd;
    [SerializeField] private float leftTriggerValue;    
    [SerializeField] private float rightTriggerValue;
    [SerializeField] private Vector3 startPos;
    [SerializeField] private Vector3 offset;
    [SerializeField] private bool isIndexTriggerDown;
    [SerializeField] public float launch_speed = 1000.0f;
    [SerializeField] public GameObject right_controller_prefab;
    [SerializeField] public GameObject left_controller_prefab;
    [SerializeField] public GameObject rocket;
    [SerializeField] public AudioSource wind;

    [SerializeField] public GameObject parachute;
   
    
    /////////////////////////////////////////////////////////
    // These are for the game mechanism.
    public ParkourCounter parkourCounter;
    public string stage;
    public SelectionTaskMeasure selectionTaskMeasure;
    private float NextFire = 0;
    private Rigidbody rb;
    void Start()
    {
        // Init some values for later use 
        rb = GetComponent<Rigidbody>();
        rb.freezeRotation = true;
        parachute.SetActive(false);
        wind.volume = 1.0f;
    }

    void Update()
    {
        
        ////////////////////////////////////////////////////////////////////////////////////////////////////
        // Please implement your LOCOMOTION TECHNIQUE in this script :D.
        
        leftTriggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryIndexTrigger, leftController); 
        rightTriggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryIndexTrigger, rightController);
        // Wind sound when moving to reduce cyber sickness
        if (rb.velocity.magnitude >= 1 && !wind.isPlaying)
        {
            
            wind.Play();
        }
        else if (wind.isPlaying && rb.velocity.magnitude <= 1)
        {
         
            wind.Stop();
            
            
        }
        else
        {
            
        }
        // Parachute technique with hands over head 
        if (hmd.transform.position.y <= left_controller_prefab.transform.position.y 
            && hmd.transform.position.y <= right_controller_prefab.transform.position.y 
            && rb.velocity.y < 0.2)
        {
            parachute.SetActive(true);
            rb.drag = 1.5f;
        }
        else
        {
            parachute.SetActive(false);
            rb.drag = 0;
        }
        
        
        if (rightTriggerValue >= 0.95f && (Time.time > NextFire)) // Fire with right trigger
        {
            NextFire = Time.time + .5f;
            LaunchRocket(right_controller_prefab);
        }
        
        if (leftTriggerValue >= 0.95f && (Time.time > NextFire)) // Fire with left trigger
        {
            NextFire = Time.time + .5f;
            LaunchRocket(left_controller_prefab);
        }
        // Original Game mechanic
        if (OVRInput.Get(OVRInput.Button.Two) || OVRInput.Get(OVRInput.Button.Four)) 
        {
            if (parkourCounter.parkourStart)
            {
                this.transform.position = parkourCounter.currentRespawnPos;
            }
        }
    }


    // Function that launches rocket from a given hand 
    void LaunchRocket(GameObject hand)
    {
        GameObject projectile = Instantiate(rocket, 
            hand.transform.position , 
            hand.transform.rotation);
       
        
    }
    ...
}
{{< / highlight >}}
This Code does not explain itself thats why I tried to come up with a short form explanation in bullet points.

Here I will give a short explanation of every variable I added to the locomotion script :
 * right/left_controller_prefab : Are Serialized Fields to get the current position of the controllers in world space, so that
 I can initiate the rockets from them. 
 * rocket : This provides the script with rocket prefab .
 * wind : Is a Audio Source to play a wind sound that prevent cyber sickness.
 * NextFire : Is a counter that checks if the fire button has been pressed in the last seconds.
 * Parachute : Is a Prefab for a Parachute model.
 
The functionalities that were added there :
 * deploying a Parachute when you move your hands above your head while falling and slowing the fall , this was also added to prevent cyber sickness a bit more. (See Line 63-74)
 * locking the rotation so that the player camera rig does not get tipped over by the explosions of the rockets. I also tried a straightening mechanism, but it caused even more cyber sickness then only locking the rotation. 
 * launching rockets (See 77-87) with the press of the primary trigger button on the VR controller.
 * adding a wind sound while moving, prevent cyber sickness as much as the locomotion method allows.
 * haptic feedback when launching a projectile, due not being super light and by that not being really interesting for representing a rocket launching. 

 ### Problems 
 Some problems I encountered here were again finding out what Unity already provides. Before adding the NextFire lock it also happened that a projectile would collide with another. This was of course fixed by adding this NextFire timer to the rocket shooting process. The process of trying to prevent cyber sickness with this kind of locomotion is quite hard as well due explosions being quite harsh in moving the player.


 # Future 
 I also tried about implement different kind of rockets that all had different kind of strengths of force pushing the player but afterwards decided against it due to finding no satisfying technique for such or editing the force value of the current rocket. One other thing that could be implemented would be to exchange the models for the controllers with real "Rocket Launcher" model with a reload animation. A good version for haptic feedback would be nice to have in the future too. This could lead to even more immersive game play.

 
After implementing everything the locomotion method looks the following way: 

{{< youtube w1whl6D1MKo>}}

