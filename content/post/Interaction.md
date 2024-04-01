+++
title = 'Interaction'
date = 2024-04-01T01:27:56+02:00
draft = false
summary = "Implementation of the interaction mechanic"
+++
# Introduction
In the following text I will describe the interaction method I implemented. What I wanted to do here is to stay in the theme of rockets for my interaction method as well so I sated generating a Prefab for another kind of projectile again. This projectile should be able to interact with objects in the scene. 
## Projectile 
Quite similar to the locomotion technique the I created a prefab for this projectile again so that I could initiate it on a button press again. This the looked the following way : 

{{<img1 src = "../../post/grab_prefab.png">}}

The grab with projectile script here allows the script to have the projectile to have as owner as well as a selected object. The owner variable here allows me to send information from the projectile game object to the owner game object, as well as moving it to that designated hand. The selected variable is set to the collied object after a collision with it. These functionalities are represented in the following code.
{{< highlight csharp "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
  void Start()
    {
       gameObject.transform.localScale /= 10;
       
    }

    // Update is called once per frame
    void Update()
    {
        
        if (returning)
        {
            transform.position = Vector3.MoveTowards(transform.position, 
                this.owner.transform.position, 0.7f);
            if (Vector3.Distance(transform.position,this.owner.transform.position) < 0.8f)
            {
                
                returning = false;
                this.gameObject.GetComponent<MeshRenderer>().enabled = false;
                this.gameObject.GetComponent<Collider>().enabled = false;
            }
        }
    }

    public void getSelected(GameObject thing)
    {
        
        if (selected == null)
        {
            
            thing =  new GameObject("Nothing");
        }
        else
        {
            thing =  selected;
        }
    }
    public void setOwner(GameObject hand)
    {
        this.owner = hand;
        gameObject.GetComponent<Rigidbody>().AddForce(hand.transform.forward * 30,ForceMode.Impulse);
    }

    void returnToOwner()
    {
        if (this.owner != null)
        {
            returning = true;
        }
    }
    // Code 1 : 
    private void OnCollisionEnter(Collision other)
    {
        if (other == null)
        {
            selected = null;
            something_selected = false;
            returnToOwner();
        }
        else if (this.owner != null && other.gameObject != this.owner && !returning)
        {
            selected = other.gameObject;
            something_selected = true;
            
            returnToOwner();
        }
    }
{{{{< / highlight >}}}}

As you can see at the "Code 1" comment the projectile is then returned to the owner game object. After returning the game object is not deleted here because we will need information about it in my grab mechanic even after it being returned. 

## Gabbing 
The interaction mechanic is then implemented in the MyGrab.cs file. For the grabbing projectile the most important part is the update part.
{{< highlight csharp "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
 void Update()

    {
        triggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryHandTrigger);
        if (triggerValue > 0.95 && (Time.time > NextFire))
        {
            NextFire = Time.time + .8f;
            shoot_projectile();
        }

        if (OVRInput.Get(OVRInput.Button.Three)) // x Button for making the object stationary again or not 
        {
            stationary = !stationary;
        }

        // projectile mechanic
        if (current_rocket != null)
        {
            if (current_rocket.GetComponentInChildren<grab_with_prjectile>().something_selected )
            {
                selectedObj = current_rocket.GetComponentInChildren<grab_with_prjectile>().selected;
                // selecting and grabbing the T object 
                if (selectedObj.gameObject.CompareTag("objectT"))
                {
                    isInCollider = true;
                    if (!stationary)
                    {
                        //making the object rotate in the same way the right controller is in world space 
                        selectedObj.transform.position = Vector3.MoveTowards(selectedObj.transform.position,
                            right_controller.transform.position, 0.3f);
                        
                        Quaternion rot = right_controller.transform.rotation;
                        selectedObj.transform.rotation = rot;
                    } else if (stationary)
                    {
                        //translation with left thumb stick and right controller input forward 
                        float input = OVRInput.Get(OVRInput.Axis2D.PrimaryThumbstick)[0];
                        if (input != 0)
                            selectedObj.transform.position += right_controller.transform.forward * input * 0.02f;
                        isInCollider = false;

                    }
                    else
                    {
                        isInCollider = false; }
                    
                    
                

                    
                }
                // stating the task 
                else if (selectedObj.gameObject.CompareTag("selectionTaskStart"))
                {
                    selectionTaskMeasure.isTaskStart = true;
                    if (!start)
                    {
                        
                        start = true;
                        if (!selectionTaskMeasure.isCountdown)
                        {
                            selectionTaskMeasure.isTaskStart = true;
                            selectionTaskMeasure.StartOneTask();
                        }
                    }
                }
                // ending the task 
                else if (selectedObj.gameObject.CompareTag("done"))

                {       Destroy(current_rocket);
                        done = false;
                        start = false;
                        stationary = false;
                        selectionTaskMeasure.isTaskStart = false;
                        
                        
                        selectionTaskMeasure.EndOneTask(); 
                        selectedObj = null;
                    
                }
            }
             
        }
    } 
{{< / highlight >}}
The controls for the shooting are: 
* left thumb button for projectile shooting 
* "x" for making an object stationary in space after rotation in hand 
* the left thumb stick together with the direction the right controller is pointing for the translation 


## Problems 
Some bugs I encountered here are, the game crashing after stating or ending the task because the game object that was in the selected does not exist any more. Another hiccup that is worth mentioning is that the task stated multiple times before introducing the start bool variable. I also had to remove the Grab mechanic from the right controller because it was always interfering with the accuracy of the selection method. This was due to the script always being call when one of the thumb buttons had been pressed ( both hands shooting). This could also have been fixed by providing the line 3 with of the prior code segment with the current controller.

{{<youtube e8jWBLISXuo>}}
# Interaction Method Video 


{{<youtube nA2ipZqMGcA>}}

{{<youtube SNOnb72FuD4>}}