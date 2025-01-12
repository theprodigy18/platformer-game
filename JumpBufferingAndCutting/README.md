# Jump Buffering and Jump Cutting in Unity 🎮  

This sections demonstrates the implementation of **Jump Buffering** and **Jump Cutting**, two advanced mechanics that enhance the responsiveness and control of jumping in platformer games.  

---

## 📝 **Introduction**  
Jumping is a core mechanic in most platformer games. Implementing features like **Jump Buffering** and **Jump Cutting** improves the player experience by making jumps feel more fluid and responsive.  

### **Jump Buffering**  
- Ensures that a jump input is not missed if pressed just before the character lands.  
- Allows for a more forgiving jump mechanic.  

### **Jump Cutting**  
- Gives players the ability to cut short their jump by releasing the jump button.  
- Provides precise control over jump height.  

---

## 🚀 **Code Overview**  

The provided code showcases how to implement both mechanics:  
```csharp
private Rigidbody2D rb;
private bool isCuttingJump = false;
private bool canCuttingJump = true;
private bool isGrounded = true;
private float jumpImpulse = 10f; // you can make this or other variable field to be SerializeField/public so you can edit it in inspector and have tracking on it.
```
Those are variable used to do both mechanism.

### **With New UnityInputSystem**
```csharp
public void OnJump(InputAction.CallbackContext context)
{
  if (context.performed)
  {
    StartCoroutine(Jump());
  }
  else if (context.canceled && !isCuttingJump && rb.linearVelocityY >= 0 && canCuttingJump) // we need to check if the jumping states is not in falling states by using linearVelocityY on Rigidbody2D
  {
    isCuttingJump = true; // set the cutting jump to be true so the player cant do cut jump multiple time in air, its can causes character stopping in mid-air for period times.
    StartCoroutine(CutJump());
  }
}
```
You can assign this function to your player input component in inspector or assign it to generated C# class inputActions in OnEnable function.
### **With KeyCode**
```csharp
private void Update()
{
  if (Input.GetKeyDown(KeyCode.Space)) // For example use space bar for jump
  {
    StartCoroutine(Jump());
  }
  else if (Input.GetKeyUp(KeyCode.Space)) && !isCuttingJump && rb.linearVelocityY >= 0 && canCuttingJump) // we need to check if the jumping states is not in falling states by using linearVelocityY on Rigidbody2D
  {
    isCuttingJump = true; // set the cutting jump to be true so the player cant do cut jump multiple time in air, its can causes character stopping in mid-air for period times.
    StartCoroutine(CutJump());
  }
}
```
Using update method so it can check for every frame, my suggestion is use new unity input system since its give you more flexibility.
Those two methods on top basically are same. Firstly its check if Jump input is got pressed, then run jump coroutine method. The second is if the jump input got released, then run the cut jump coroutine method.

### **Core Logic**
```csharp
private IEnumerator Jump()
{
    float jumpBufferingTime = 0.25f; // set the input pressed time tolerance(bigger number will make you can press jump input way before it land on the ground) 
    while (jumpBufferingTime > 0.0f) // loop until the time tolerance reached zero
    {
        if (isGrounded) // if character is touching the ground 
        {
            canCuttingJump = true; // set the canCuttingJump to true, so player can make jump cutting 
            animator.SetTrigger("jump"); // set the animation if you have
            yield return new WaitForSeconds(0.05f); // this delay is for sync with animation, delete this line if you want no delay
            rb.linearVelocityY = jumpImpulse; // Apply jump force 
            yield return new WaitForSeconds(0.25f); // you can play around with this number base on how long your jump states before reaching your maximum velocity
            canCuttingJump = false; // Disable jump cutting because its already reached the max jump height
            yield break; // don't forget to break the loop
        }

        jumpBufferingTime -= Time.deltaTime; // reduce time tolerance with Time.deltaTime so it have fixed number base on time.
        yield return null;
    }
}
private IEnumerator CutJump()
{
    yield return new WaitForSeconds(0.2f); // Allow jump slightly longer so the jump is not to short 
    rb.linearVelocityY = 0; // Stop upward velocity for so the jump height will be cut and start falling
}
```
---

## 🏁 **Conclusion**  

By implementing **Jump Buffering** and **Jump Cutting**, you can significantly improve the responsiveness and precision of your platformer game's jump mechanics. These features enhance the overall player experience by:  

1. Allowing jumps to register even if the input timing is slightly off.  
2. Providing greater control over jump height, making gameplay more engaging and skill-based.  

With a small amount of additional logic, these mechanics can make your game feel polished and professional. Experiment with the parameters to match your desired gameplay style and create a truly enjoyable platformer experience!  

Happy developing! 🚀  
