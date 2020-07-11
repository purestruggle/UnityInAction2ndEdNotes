# Unity in Action Chapter 2 Notes

### Access Modifiers (C# Programming) <- Basically Scope stuff 
* `Public`: The type or member can be accessed by any other code in the same 
  assembly or another assembly that references it.
* `Private`: The type or member can be accessed only by code in the same 
  class or struct.

`*=`: Multiplies the value of a variable or property by the value of an 
expression the result to variable or property

---
```c 
public void Start();
// Called when an object is active. First thing to run
```

---
```c 
public void Update()
// Called and updated every frame
```

---
```
GameObjects are fundamental objects in Unity that represent characters, props
and scenery. They act as containers for Components which implement real
functionality.
```
https://docs.unity3d.com/Manual/class-GameObject.html

---
```
Components are the nuts and bolts of objects and behaviors in game.
The functional parts of a GameObject.
For example: the Transform component of a GameObject dictates where the 
object is located (position, rotation, and scale). A light component would 
give the GameObject light, etc.
```
https://docs.unity3d.com/Manual/Components.html

---
```c
Input.GetAxis("Horizontal");
Input.GetAxis("Vertical");
// Interface into the Input system.
// Returns the value of the virtual axis
// identified by axisName.
// The value will be in the range -1...1 for
// keyboard and joystick input devices.
```
https://docs.unity3d.com/ScriptReference/Input.GetAxis.html

---
```c
Vector3 movement = new Vector3(
            deltaX,
            0,
            deltaZ
        );
// Creates a new vector with given x, y, z components.
```
https://docs.unity3d.com/ScriptReference/Vector3.html

---
```c
Vector3.ClampMagnitude(movement, speed);
// Returns a copy of vector with its magnitude clamped to maxLength.
```
https://docs.unity3d.com/ScriptReference/Vector3.ClampMagnitude.html

---
```c
Time // Interace to pull time information from Unity
Time.deltaTime 
// The completion time in seconds since the last frame (Read Only)
// This property provides the time between the current and previous frame
```
https://docs.unity3d.com/ScriptReference/Time-deltaTime.html

---
```c
transform // Position, rotation, and scale of an object.
transform.TransformDirection(Vector3 direction);
transform.TransformDirection(float x, float y, float z);
// Transforms direction x, y, z (Vector3) from local space to world space. 
// This operation is not affected by scale or position of the transform. 
// The returned vector has the same length as direction.
```

---
Prefixing variable with underscore i.e. _charController is a way of indicating 
variable scope.
https://stackoverflow.com/questions/549606/what-does-variable-names-beginning-with-mean

---
```c
float speed = 6.0f; 
// Precision: 7 digits
// f is used to denote that this variable value is of type float
```
https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types

---
```c
private CharacterController 
// A CharacterController allows you to easily do movement constrained by 
// collisions without having to deal with a rigidbody.

// A CharacterController is not affected by forces and will only move when you 
// call the Move function. It will then carry out the movement but be 
// constrained by collisions.
``` 
https://docs.unity3d.com/ScriptReference/CharacterController.html

---
```c
CharacterController.Move(Vecter3 motion);
// Supplies the movement of a GameObject with an attached CharacterController 
// component.
// Moves the GameObject in the given direction.
```
https://docs.unity3d.com/ScriptReference/CharacterController.Move.html

---
```c
GetComponent<CharacterController>();
// Returns the component of Type type if the game object has one attached,
// null if it doesn't.
```
https://docs.unity3d.com/ScriptReference/Component.GetComponent.html
https://docs.unity3d.com/Manual/GenericFunctions.html

---
```c
transform.Translate(Vector3 translation);
transform.Translate(Vector3 translation, Space relativeTo = Space.Self)
// Moves the transform in the direction and distance of translation.
// If relativeTo is left out or set to Space.Self the movement is applied 
// relative to the to the transform's local axes. (the x, y and z axes shown
// when selecting the object inside the Scene View)
```
https://docs.unity3d.com/ScriptReference/Transform.Translate.html

---
```c
Debug.Log("Message");
Debug.Log(Variable)
// When line runs it prints message to the command line or can be used to track
// current state of variable i.e. numbers being added in etc.
```
https://docs.unity3d.com/ScriptReference/Debug.Log.html




