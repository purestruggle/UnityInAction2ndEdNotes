# Unity in Action Chapter Three Notes

## Shooting via Raycasts

### Summary:
* A ray is an imaginary line projected into the scene.
* For both shooting and sensing obstacles, do a raycast with that line.
* Making a character wander around involves basic AI.
* New objects are spawn by instantiating prefabs.
* Coroutines are used to spread out functions over time.
  
Previous chapter we got movement to work, now we are going to add shooting to 
the mix.

Ray = is an imaginary or invisible line in the scene that starts at a point of 
origin and extends out in a specific direction. 

Raycasting = when you create a ray and then determine what intersects that ray.
i.e. bullet from gun.

### Commented Code: 

```c
//RayShooter.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class RayShooter : MonoBehaviour
{
    private Camera _camera;
    // Start is called before the first frame update
    void Start()
    {
        // Start of script the _camera private Camera component is called. We
        // can then modify the Camera object
        // https://docs.unity3d.com/ScriptReference/Component.GetComponent.html 
        _camera = GetComponent<Camera>();

        // Uses the Cursor API to lock the cursor to the center of the view. The
        // cursor is invisible in this state according to documentation so I'm
        // unsure why the book makes you define Cursor visibility below.
        // https://docs.unity3d.com/ScriptReference/Cursor-lockState.html
        Cursor.lockState = CursorLockMode.Locked;
        
        // Cursor API to make the cursor not appear when locked to the center of
        // the screen.
        Cursor.visible = false;
    }

    // OnGui is called for rendering and handling GUI events. OnGui code defines
    // the 2D coordinates for the display.
    void OnGUI() 
    {
        // Creating the dimensions for the Rectangle that will be the GUI Label
        // to form the crosshair.
        int size = 12;
        float posX = _camera.pixelWidth/2 - size/4;
        float posY = _camera.pixelHeight/2 - size/2;
        
        // Uses rectangle dimensions above to form the 2D coords to place the
        // crosshair or in this case the asterisk.
        GUI.Label(new Rect(posX, posY, size, size), "*");
    }
    // Update is called once per frame
    void Update()
    {
        // If the primary button (left mouse button) is pressed down then
        // execute the if statement.
        if (Input.GetMouseButtonDown(0)) 
        {
            Vector3 point = new Vector3(
                _camera.pixelWidth/2, 
                _camera.pixelHeight/2,
                0);
            Ray ray = _camera.ScreenPointToRay(point);
            RaycastHit hit;

            //If the casted ray hits something it returns the hit information.
            if (Physics.Raycast(ray, out hit)) 
            {   
                //Retrieves the object that the ray hit.
                GameObject hitObject = hit.transform.gameObject;

                //target variable stores the GetComponent 
                ReactiveTarget target = hitObject.GetComponent<ReactiveTarget>();
                if (target != null) {
                    // Call a function of the target.
                    target.ReactToHit();
                } else {
                    // Starts are Coroutine that is the function that is defined
                    // below. Will continue to run till finished then continue
                    // running frame.
                    StartCoroutine(SphereIndicator(hit.point));
                }
                
            }
        }
    }
    // new Method that is called within Update function every frame. If the
    // Physics.Raycast logic is accessed. Accepts a vector position. IEnumerator
    // is a concept that is tied with StartCoroutine. Forms a sphere at the
    // target location and destroys it. Hands back program flow the Update to
    // continue the frame.
    private IEnumerator SphereIndicator(Vector3 pos) 
    {
        // Creates a basic sphere shape Game Object  with primitive mesh
        // renderer and collider. 
        GameObject sphere = GameObject.CreatePrimitive(PrimitiveType.Sphere);

        // Updates the sphere GameObjects position in the world with the vector3
        // pos passed into the function.
        sphere.transform.position = pos;

        // yield controls the length of the pause to wait; in this case we are
        // waiting 1 second.
        yield return new WaitForSeconds(1);

        // Destroy the GameObject we created.
        Destroy(sphere);
    }
}

```

```c
//ReactiveTarget.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ReactiveTarget : MonoBehaviour
{   
    // Function that is called when GetComponent returns a GameObject.
    public void ReactToHit()
    {
        // Pull the WanderAI component into this function.
        WanderingAI behavior = GetComponent<WanderingAI>();
        if (behavior != null)
        {
            // Set the bot to dead by using WanderAI component and the
            // SetAlive function.
            behavior.SetAlive(false);
        }
        // Calls private function below to tranform the GameObject to react to
        // the RayCast.
        StartCoroutine(Die());
    }

    private IEnumerator Die()
    {
        // Refers to the GameObject that the script is recieving from the
        // raycast. 
        this.transform.Rotate(-75, 0, 0);

        //Waits for 1.5 seconds.
        yield return new WaitForSeconds(1.5f);

        // Destroys the GameObject that raycast hits after waiting the above
        // seconds. 
        Destroy(this.gameObject);
    }
}

```

```c
//WanderAI.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


public class WanderingAI : MonoBehaviour
{
    public float speed = 3.0f;
    public float obstacleRange = 5.0f;
    private bool _alive;
    [SerializeField] private GameObject fireballPrefab;
    private GameObject _fireball;
    void Start() 
    {
        _alive = true;
    }
    void Update()
    {
        if (_alive)
        {
            // Moves along the Z axis at a speed of frame rate-independent
            // movement. 
            transform.Translate(0, 0, speed * Time.deltaTime);
        }
        // Drawing a ray from the character and pointing it at a length forward.
        Ray ray = new Ray(transform.position, transform.forward);
        RaycastHit hit;

        // Do raycasting with a circumference around the ray that is being cast
        // forward. 
        if (Physics.SphereCast(ray, 0.75f, out hit))
        {
            GameObject hitObject = hit.transform.gameObject;
            if (hitObject.GetComponent<PlayerCharacter>())
            {
                // Check if a fireball GameObject already exists in the scene.
                if (_fireball == null)
                {
                    // Copies the prefab (fireball) to the variable and creates
                    // a copy into the scene.
                    _fireball = Instantiate(fireballPrefab) as GameObject;
                    
                    // Tranform fireball GameObject right infront of the enemy
                    // GameObject and push it forward 1.5 points.
                    _fireball.transform.position = 
                        transform.TransformPoint(Vector3.forward * 1.5f);
                    _fireball.transform.rotation = transform.rotation;

                     
                    
                }
                // If the distance on raycast is less than obstacleRange it will
                // execute the else if code.
                else if (hit.distance < obstacleRange)
                {
                    // Generate random angle value between -110 and 110 to
                    // insert into rotate transform
                    float angle = Random.Range(-110, 110);
                    transform.Rotate(0, angle, 0);
                }
            }
            // If the hit distance on raycast is less than obstacleRange it will
            // execute the if code.
            if (hit.distance < obstacleRange)
            {
                // Declare a float variable for a angle that will randomize a
                // value for the y value to insert into the tranform below.
                float angle = Random.Range(-110, 110);
                transform.Rotate(0, angle, 0);
            }
        } 
    }
    // Public function for keeping track of the enemies alive state. 
    public void SetAlive(bool alive)
    {
        _alive = alive;
    }
}
```

```c
//SceneController.cs
using System.Collections;
using UnityEngine;

public class SceneController : MonoBehaviour
{
    [SerializeField] private GameObject enemyPrefab;
    private GameObject _enemy;

    // Update is called once per frame
    void Update()
    {
        // Check if an Enemy GameObject already exists in the scene.
        if (_enemy == null)
        {
            // Copies the prefab to the variable and creates a copy into the
            // scene. 
            _enemy = Instantiate(enemyPrefab) as GameObject;

            // Moving Enemy GameObject to specified vector on the gamemap.
            _enemy.transform.position = new Vector3(0, 1, 0);

            // Pick random value between 0-360 and insert into y value rotation
            // of below tranform.
            float angle = Random.Range(0, 360);
            _enemy.transform.Rotate(0, angle, 0);
        }
    }
}

```

```c
//PlayerCharacter.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerCharacter : MonoBehaviour
{
    private int _health;
    
    // Start is called before the first frame update
    void Start()
    {
        // Set player health.
        _health = 5;
    }

    // Making stuff hurt the player.
    public void Hurt(int damage) 
    {
        // On hit subtract damage from health.
        _health -= damage;
        Debug.Log("Health: " + _health);
    }
}

```

```c
//Fireball.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Fireball : MonoBehaviour
{
    public float speed = 10.0f;
    public int damage = 1;

    // Update is called once per frame
    void Update()
    {
        // Makes the fireball fly foward along the z axis.
        transform.Translate(0, 0, speed * Time.deltaTime);    

    }

    // When a GameObject collides with another GameObject, this function
    // triggers. 
    void OnTriggerEnter(Collider other) 
    {
        // Setting player variable to component of PlayerCharacter.
        PlayerCharacter player = other.GetComponent<PlayerCharacter>();

        // Check if player variable isn't null and has collided with player
        // print "Player hit"
        if (player != null)
        {
            player.Hurt(damage);
            Debug.Log("Player hit");
        }

        //Destroys the GameObject referenced in this function.
        Destroy(this.gameObject);    
    }
}

```

### Notes on Unity Calls:

```c
Camera.ScreenPointToRay(Vector3 pos);
Camera.ScreenPointToRay(Vector3 pos, Camera.MonoOrStereoscopicEye eye);
```
Returns a ray going from camera through a screen point.
Resulting ray is in world space, starting on the near plane of the camera and 
going through position's (x,y) pixel coordinates on the screen.
https://docs.unity3d.com/ScriptReference/Camera.ScreenPointToRay.html

```c
Physics.Raycast()
```
Casts a ray, from point `origin`, in direction `direction`, of length
`maxDistance`, against all colliders in the Scene.
https://docs.unity3d.com/ScriptReference/Physics.Raycast.html

---
```c
// Importing the different libraries needed for the scripts. Most will need the
// below scripts
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
```

---
```c
// Defining the class for the C# script.
public class RayShooter : MonoBehaviour
```

---
```c
// Defines the camera through which the player views the world.
private Camera _camera;
```
https://docs.unity3d.com/ScriptReference/Camera.html

---
```c
// Start of script the _camera private Camera component is called. We can then
// modify the Camera object
_camera = GetComponent<Camera>();
```
https://docs.unity3d.com/ScriptReference/Component.GetComponent.html


---
```c
// If the player presses down the left mouse button it will execute the
// following if statement logic.
if (Input.GetMouseButtonDown(0)) {
```
https://docs.unity3d.com/ScriptReference/Input.GetMouseButtonDown.html

---
```c
    // Creating new Vector3 point with Camera.pixelWidth for x coords, 
    // Camera.pixelHeigh for y coords, and 0 for z coords.
    // Dividing both those in half will give us the center of the 2D space made
    // up of both pixelWidth and pixelHeight. Both are readonly values. 
    Vector3 point = new Vector3(
        _camera.pixelWidth/2, 
        _camera.pixelHeight/2,
        0);
```
https://docs.unity3d.com/ScriptReference/Camera-pixelWidth.html
https://docs.unity3d.com/ScriptReference/Camera-pixelHeight.html

---
```c
    // Creats a ray with the origin being the camera and through the point;
    // the point in this case being the center of the camera created by divding
    // camera height and width in half.
    Ray ray = _camera.ScreenPointToRay(point);
```
https://docs.unity3d.com/ScriptReference/Ray-ctor.html
https://docs.unity3d.com/ScriptReference/Camera.ScreenPointToRay.html

---
```c    
    // Used to contain information back from a raycast.
    RaycastHit hit;
```
https://docs.unity3d.com/ScriptReference/RaycastHit.html

---
```c
    // Physics.Raycast returns a boolean if the ray intersects with a Collider.
    // "out hit" returns the information back from the raycast.
    if (Physics.Raycast(ray, out hit)) {
        // Logs message using RaycastHit variable and the point in world space 
        // the ray hits the collider
        Debug.Log("Hit " + hit.point);
        }
```
https://docs.unity3d.com/ScriptReference/Physics.Raycast.html
https://docs.unity3d.com/ScriptReference/RaycastHit-point.html

Prefabs are a full fleshed out game object (with components already attached and
set up) that doesn't exist in any specific scene, but rather exists as an asset
that can be copied into any scene.

Issues with "never assigned to, and will always have its default value null"
warning:
https://answers.unity.com/questions/60461/warning-cs0649-field-is-never-assigned-to-and-will.html

