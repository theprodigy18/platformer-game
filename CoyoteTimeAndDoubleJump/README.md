# Coyote Time and Double Jump in Unity 🎮

This section demonstrates the implementation of **Coyote Time** and **Double Jump**, two advanced mechanics that enhance player control and forgiveness in platformer games.

---

## 📝 **Introduction** 
Jumping is a critical mechanic in platformer games. Implementing **Coyote Time** and **Double Jump** adds depth and polish to gameplay.

### **Coyote Time**  
- Allows players to jump for a short time after leaving a platform.
- Prevents frustrating missed jumps due to strict timing. 

### **Jump Cutting**  
- Grants players an additional jump while in mid-air.  
- Adds an exciting layer of mobility and strategy to the game.

---

## 🚀 **Code Overview** 

The provided code demonstrates how to implement both mechanics effectively (Still using **Jump Buffering** and **Jump Cutting** from previous tutorial):  
```csharp
private Rigidbody2D rb;
private bool isCuttingJump = false;
private bool canCuttingJump = true;
private bool isGrounded = true;
private float jumpImpulse = 10f; // you can make this or other variable field to be SerializeField/public so you can edit it in inspector and have tracking on it.
private float coyoteTime = 0.2f;
private float coyoteTimeBuffer;
private bool canDoubleJump = true;
```
These variables manage both Coyote Time and Double Jump. The Input implementation still same from previous tutorial(**Jump Buffering** and **Jump Cutting**).

### **Core Logic**
```csharp
private void Awake()
{
    coyoteTimeBuffer = coyoteTime; // initialize the timer at the start
}
private void Update()
{
    if (!isGrounded)
    {
        coyoteTimeBuffer -= Time.deltaTime; // start counting if character is not grounded/in mid-air
    }
    if (IsGrounded && coyoteTimeBuffer < coyoteTime)
    {
        coyoteTimeBuffer = coyoteTime; // reset timer if character touch the ground 
    }
    if (IsGrounded)
    {
        if (!canDoubleJump)
        {
            canDoubleJump = true; // reset doubleJump if character touch the ground, so player can do double jump again after it
        }
        if (isAlreadyCutJump)
        {
            isAlreadyCutJump = false; // i forgot to show how to reset cut jump bool on the previous tutorial
        }
    }
}

private IEnumerator Jump()
{
    if (!isGrounded && (coyoteTimeBuffer <= 0f || canCuttingJump) && canDoubleJump) // if it from falling then check if the coyoteTime is reached below zero, if it from jump then check if player canCuttingJump(so if the player start falling after jump it still can do secondJump due to coyoteTime already reached zero, this was my method to prevent double jumping before coyote time zero if it not already jump)
    {
        StartCoroutine(SecondJump()); // Start the second Jump
        yield break; 
    }
    float jumpBufferingTime = 0.25f; // set the input pressed time tolerance(bigger number will make you can press jump input way before it land on the ground) 
    while (jumpBufferingTime > 0.0f) // loop until the time tolerance reached zero
    {
        if (isGrounded || coyoteTimeBuffer > 0) // if character is touching the ground 
        {
            coyoteTimeBuffer = 0; // reset timer immediately
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
private IEnumerator DoSecondJump()
{
    animator.SetTrigger("secondJump"); // set the animation if you have, you can add delay to match your animation
    canDoubleJump = false; // set to false so player only can do second jump once before touching the ground
    rb.linearVelocityY = jumpImpulse - 2; // you can play with this number, i do substract the normal jumpImpulse so the second jump will not have same vertical boost.
    yield return null;
}
```
---

## 🏁 **Conclusion**

By implementing **Coyote Time** and **Double Jump**, you create a more engaging and player-friendly platforming experience:

1. **Coyote Time** prevents missed jumps and enhances player forgiveness.
2. **Double Jump** adds a layer of strategy and excitement to movement mechanics. 

These mechanics ensure smooth gameplay and help create a polished, enjoyable platformer. Adjust the parameters to suit your game's unique style and design.

Happy coding! 🚀 
