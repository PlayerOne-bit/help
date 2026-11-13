# Unity C# Scripting Tutorial

## A beginner-friendly guide for Unity 6

Learn how to write C# scripts, connect them to GameObjects, read player
input, move objects, detect collisions, and build a small playable
project.

> **Goal:** By the end, you will understand the basic structure of Unity
> scripts and be able to create a simple 3D player controller and
> collectible system.

------------------------------------------------------------------------

# 1. What Is a Unity Script?

Unity uses **C#** to control how objects behave in a game.

Examples: - Move a player. - Make an enemy follow the player. - Open a
door when a button is pressed. - Collect coins. - Update health and
score. - Play animations and sounds.

A script is a C# file that can be attached to a GameObject.

## Important Unity terms

  -----------------------------------------------------------------------
  Term                                Meaning
  ----------------------------------- -----------------------------------
  GameObject                          An object in your scene, such as a
                                      player, enemy, or camera

  Component                           A feature attached to a GameObject

  Transform                           Controls position, rotation, and
                                      scale

  Script                              Your custom C# component

  Inspector                           Where you view and edit
                                      object/component properties

  Scene                               The level or environment you are
                                      building

  Prefab                              A reusable GameObject template
  -----------------------------------------------------------------------

# 2. Create Your First Script

1.  Open your Unity project.
2.  In the **Project** window, right-click.
3.  Select **Create → Scripting → MonoBehaviour Script** (the menu
    wording may vary by Unity version).
4.  Name it `FirstScript`.
5.  Double-click the script to open it in your code editor.

Example:

``` csharp
using UnityEngine;

public class FirstScript : MonoBehaviour
{
    void Start()
    {
        Debug.Log("Hello, Unity!");
    }

    void Update()
    {

    }
}
```

## Understanding the code

-   `using UnityEngine;` gives access to Unity features.
-   `public class FirstScript` defines your script.
-   `: MonoBehaviour` allows Unity to use it as a component.
-   `Start()` runs once when the component becomes enabled and starts.
-   `Update()` runs once per frame while the component is enabled.
-   `Debug.Log()` prints a message in the Console.

## Attach the script

1.  Select a GameObject in the Hierarchy.
2.  Drag `FirstScript` from the Project window into the Inspector.
3.  Press **Play**.
4.  Open **Window → General → Console** if you cannot see the message.

**Important:** The C# class name and file name should match. For
example, `FirstScript.cs` should contain `public class FirstScript`.

# 3. Variables and Data Types

Variables store information.

``` csharp
using UnityEngine;

public class PlayerStats : MonoBehaviour
{
    int health = 100;
    float speed = 5.5f;
    string playerName = "Player";
    bool isAlive = true;

    void Start()
    {
        Debug.Log(playerName);
        Debug.Log(health);
        Debug.Log(speed);
        Debug.Log(isAlive);
    }
}
```

  Type       Stores                         Example
  ---------- ------------------------------ ----------------------------
  `int`      Whole numbers                  `int score = 10;`
  `float`    Decimal numbers                `float speed = 4.5f;`
  `double`   More precise decimal numbers   `double distance = 12.75;`
  `string`   Text                           `string name = "Alex";`
  `bool`     True or false                  `bool isOpen = false;`

## Public variables and the Inspector

A public variable can be edited in the Inspector.

``` csharp
using UnityEngine;

public class PlayerSettings : MonoBehaviour
{
    public float moveSpeed = 5f;
    public int maxHealth = 100;
}
```

You can change these values without editing the code.

A common alternative is `[SerializeField] private`. It keeps the
variable private in C# but still exposes it in the Inspector:

``` csharp
[SerializeField] private float moveSpeed = 5f;
```

Use private fields by default and expose only what needs to be
configured.

# 4. Methods and Parameters

A method is a reusable block of code.

``` csharp
using UnityEngine;

public class PlayerActions : MonoBehaviour
{
    void Start()
    {
        SayHello();
        TakeDamage(20);
    }

    void SayHello()
    {
        Debug.Log("Hello, player!");
    }

    void TakeDamage(int damage)
    {
        Debug.Log("Damage received: " + damage);
    }
}
```

-   `void` means the method does not return a value.
-   `TakeDamage(int damage)` receives a number as an argument.
-   `SayHello()` can be called whenever you need that action.

A method can also return a value:

``` csharp
int Add(int a, int b)
{
    return a + b;
}
```

# 5. Unity's Main Lifecycle Methods

Unity calls certain methods automatically.

  -----------------------------------------------------------------------
  Method                  When it runs            Common use
  ----------------------- ----------------------- -----------------------
  `Awake()`               When the script         Set up references
                          instance is loaded      

  `OnEnable()`            When the component      Subscribe to events
                          becomes enabled         

  `Start()`               Before the first frame  Initial setup
                          update                  

  `Update()`              Every frame             Input and frame-based
                                                  logic

  `FixedUpdate()`         At fixed time intervals Physics calculations

  `LateUpdate()`          After `Update()`        Camera follow logic

  `OnDisable()`           When the component is   Unsubscribe from events
                          disabled                

  `OnDestroy()`           When the object is      Cleanup
                          destroyed               
  -----------------------------------------------------------------------

Example:

``` csharp
using UnityEngine;

public class LifecycleExample : MonoBehaviour
{
    void Awake()
    {
        Debug.Log("Awake");
    }

    void Start()
    {
        Debug.Log("Start");
    }

    void Update()
    {
        // Runs every frame.
    }

    void FixedUpdate()
    {
        // Runs at fixed time intervals.
    }
}
```

Do not put expensive or unnecessary work in `Update()` because it can
run many times per second.

# 6. Transform: Position, Rotation, and Scale

Every GameObject has a Transform component.

``` csharp
using UnityEngine;

public class TransformExample : MonoBehaviour
{
    void Start()
    {
        transform.position = new Vector3(0, 1, 0);
        transform.rotation = Quaternion.identity;
        transform.localScale = new Vector3(2, 2, 2);
    }
}
```

A `Vector3` contains three values: X, Y, and Z.

-   X: left and right
-   Y: down and up
-   Z: backward and forward

Useful Transform operations:

``` csharp
transform.position = new Vector3(0, 0, 0);
transform.Translate(Vector3.forward * 2f);
transform.Rotate(0, 90, 0);
transform.localScale = Vector3.one;
```

`Vector3.one` means `(1, 1, 1)`.

# 7. Move an Object

For a simple demonstration, move an object using `Transform`.

``` csharp
using UnityEngine;

public class SimpleMovement : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 5f;

    void Update()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        Vector3 movement = new Vector3(horizontal, 0f, vertical);

        transform.Translate(movement * moveSpeed * Time.deltaTime, Space.World);
    }
}
```

**Note:** This example uses Unity's legacy Input Manager axes
(`Horizontal` and `Vertical`). It is useful for learning, but projects
configured for the **Input System package** should use that system
instead. You can set the active input handling in **Edit → Project
Settings → Player → Other Settings → Active Input Handling**.

## What is `Time.deltaTime`?

`Time.deltaTime` is the time in seconds since the previous frame.

Multiplying movement by `Time.deltaTime` makes movement frame-rate
independent.

Without it, an object may move faster on a computer that renders more
frames per second.

# 8. Use the New Input System

For modern Unity projects, the Input System package is recommended.

## Basic setup

1.  Open **Window → Package Manager**.
2.  Find and install **Input System** if it is not installed.
3.  Open **Edit → Project Settings → Player**.
4.  Set **Active Input Handling** to **Input System Package (New)** or
    **Both**.
5.  Restart Unity if requested.

## Example: Read keyboard input

This example uses the new Input System directly:

``` csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class NewInputExample : MonoBehaviour
{
    void Update()
    {
        if (Keyboard.current == null)
            return;

        if (Keyboard.current.spaceKey.wasPressedThisFrame)
        {
            Debug.Log("Space was pressed!");
        }

        if (Keyboard.current.eKey.wasPressedThisFrame)
        {
            Debug.Log("E was pressed!");
        }
    }
}
```

`wasPressedThisFrame` is true only on the frame the key is pressed. This
is useful for actions such as jumping, interacting, or attacking.

For larger projects, use **Input Actions** so you can configure
keyboard, controller, and other device bindings without hardcoding every
key.

# 9. Components and GetComponent

Unity objects are built from components. A script can access other
components attached to the same GameObject.

Example: change an object's color using a Renderer.

``` csharp
using UnityEngine;

public class ChangeColor : MonoBehaviour
{
    private Renderer objectRenderer;

    void Start()
    {
        objectRenderer = GetComponent<Renderer>();

        if (objectRenderer != null)
        {
            objectRenderer.material.color = Color.red;
        }
    }
}
```

`GetComponent<Renderer>()` searches the same GameObject for a Renderer
component.

Always consider that the component might be missing. Check for `null`
before using it.

## Reference another GameObject

``` csharp
using UnityEngine;

public class TargetReference : MonoBehaviour
{
    [SerializeField] private GameObject target;

    void Start()
    {
        if (target != null)
        {
            Debug.Log("Target: " + target.name);
        }
    }
}
```

Drag another GameObject into the `Target` field in the Inspector.

# 10. Physics: Rigidbody and Collisions

Use physics components when an object should respond to forces, gravity,
and collisions.

## Move a Rigidbody

1.  Create a Cube.
2.  Add a **Rigidbody** component.
3.  Attach this script.

``` csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class RigidbodyMovement : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 5f;

    private Rigidbody rb;
    private Vector3 movement;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        movement = new Vector3(horizontal, 0f, vertical).normalized;
    }

    void FixedUpdate()
    {
        Vector3 nextPosition = rb.position + movement * moveSpeed * Time.fixedDeltaTime;
        rb.MovePosition(nextPosition);
    }
}
```

This example uses the legacy input axes for simplicity. If your project
uses the new Input System, read input in `Update()` with that system and
keep Rigidbody movement in `FixedUpdate()`.

## Collision detection

Two objects can collide when they have suitable Collider components and
the physics setup allows it.

``` csharp
using UnityEngine;

public class CollisionExample : MonoBehaviour
{
    void OnCollisionEnter(Collision collision)
    {
        Debug.Log("Collided with: " + collision.gameObject.name);
    }
}
```

Common collision methods:

  Method                 Meaning
  ---------------------- ---------------------------
  `OnCollisionEnter()`   Collision begins
  `OnCollisionStay()`    Objects remain in contact
  `OnCollisionExit()`    Collision ends

## Trigger detection

A trigger detects that an object entered its area without physically
blocking it.

1.  Add a Collider to an object.
2.  Enable **Is Trigger**.
3.  Use this script:

``` csharp
using UnityEngine;

public class TriggerExample : MonoBehaviour
{
    void OnTriggerEnter(Collider other)
    {
        Debug.Log(other.name + " entered the trigger!");
    }
}
```

At least one of the interacting objects generally needs a Rigidbody for
standard 3D trigger events to be detected. For 2D physics, use
`Collider2D`, `Rigidbody2D`, and methods such as `OnTriggerEnter2D()`.

# 11. Create and Destroy Objects

You can create objects from prefabs using `Instantiate()`.

``` csharp
using UnityEngine;

public class SpawnObject : MonoBehaviour
{
    [SerializeField] private GameObject objectPrefab;
    [SerializeField] private Transform spawnPoint;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            if (objectPrefab != null && spawnPoint != null)
            {
                Instantiate(objectPrefab, spawnPoint.position, spawnPoint.rotation);
            }
        }
    }
}
```

To destroy an object:

``` csharp
Destroy(gameObject);
```

To destroy it after three seconds:

``` csharp
Destroy(gameObject, 3f);
```

# 12. Coroutines and Delays

A coroutine allows a method to pause and continue later.

``` csharp
using System.Collections;
using UnityEngine;

public class CoroutineExample : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(ShowMessage());
    }

    IEnumerator ShowMessage()
    {
        Debug.Log("Starting...");

        yield return new WaitForSeconds(2f);

        Debug.Log("Two seconds passed!");
    }
}
```

Use `IEnumerator` for coroutine methods and `yield return` to wait.

# 13. Build a Beginner Project: Collect the Coins

Create a small 3D scene where a player collects coins and increases the
score.

## Step 1: Create the scene

1.  Create a Plane and name it `Ground`.
2.  Create a Capsule and name it `Player`.
3.  Add a Rigidbody to `Player`.
4.  Freeze Rigidbody rotation on X and Z so the capsule does not tip
    over.
5.  Create a Sphere and name it `Coin`.
6.  Scale the Coin to `(0.5, 0.5, 0.5)`.
7.  Enable **Is Trigger** on the Coin's Sphere Collider.
8.  Duplicate the Coin several times and place the copies around the
    ground.

## Step 2: Create the PlayerMovement script

Create `PlayerMovement.cs` and attach it to Player.

``` csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class PlayerMovement : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 5f;

    private Rigidbody rb;
    private Vector3 movement;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        movement = new Vector3(horizontal, 0f, vertical).normalized;
    }

    void FixedUpdate()
    {
        Vector3 nextPosition = rb.position + movement * moveSpeed * Time.fixedDeltaTime;
        rb.MovePosition(nextPosition);
    }
}
```

> This sample uses the legacy Input Manager. If you selected Input
> System only, replace the input-reading part with Input System code
> from Section 8. Do not mix input systems without configuring Active
> Input Handling.

## Step 3: Create the ScoreManager script

Create an empty GameObject named `GameManager`. Create `ScoreManager.cs`
and attach it to GameManager.

``` csharp
using UnityEngine;

public class ScoreManager : MonoBehaviour
{
    public static ScoreManager Instance { get; private set; }

    private int score = 0;

    void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
    }

    public void AddScore(int amount)
    {
        score += amount;
        Debug.Log("Score: " + score);
    }
}
```

This is a simple singleton: `Instance` provides a shared reference to
the active ScoreManager. For a small learning project it is convenient,
but larger projects may benefit from explicit references or other
architecture.

## Step 4: Create the Coin script

Create `Coin.cs` and attach it to every Coin.

``` csharp
using UnityEngine;

public class Coin : MonoBehaviour
{
    [SerializeField] private int scoreValue = 1;

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            ScoreManager.Instance.AddScore(scoreValue);
            Destroy(gameObject);
        }
    }
}
```

## Step 5: Set the Player tag

1.  Select Player.
2.  At the top of the Inspector, open the Tag dropdown.
3.  Choose **Player**. If it does not exist, select **Add Tag**, create
    it, and assign it.
4.  Press Play.
5.  Move the Player into a Coin.
6.  Check the Console for the score.

**Important:** Ensure the Player has a Collider and Rigidbody, the Coin
has a trigger Collider, and the GameManager has the ScoreManager
component. If you see a null-reference error, check that the GameManager
exists in the active scene.

# 14. Common Beginner Errors

  -----------------------------------------------------------------------
  Error                               Possible solution
  ----------------------------------- -----------------------------------
  Script does not appear in Inspector Check that the file name and class
                                      name match and there are no compile
                                      errors

  `NullReferenceException`            A reference or component is
                                      missing; check it before using it

  Object does not move                Check the script is attached and
                                      enabled, and confirm the input
                                      handling setting

  Collision does not work             Check Collider components,
                                      Rigidbody setup, layers, and
                                      trigger settings

  Input is not working                Check Active Input Handling and
                                      which input API the script uses

  Changes disappear after Play mode   Unity normally does not save
                                      runtime Inspector changes when Play
                                      mode ends

  Player tips over                    Freeze unwanted Rigidbody rotation
                                      axes
  -----------------------------------------------------------------------

## Useful debugging tools

``` csharp
Debug.Log("This is a normal message");
Debug.LogWarning("This is a warning");
Debug.LogError("This is an error");
```

Read the first error in the Console. Later errors may be consequences of
the first one.

# 15. Good Scripting Practices

-   Use clear names such as `playerHealth`, `moveSpeed`, and
    `CollectCoin()`.
-   Keep one main responsibility per script.
-   Use `[SerializeField] private` for Inspector settings when possible.
-   Avoid repeating the same code; create reusable methods.
-   Check references for `null`.
-   Use `Time.deltaTime` for frame-based movement.
-   Use `FixedUpdate()` for Rigidbody physics operations.
-   Do not put everything inside one giant script.
-   Use comments to explain why complicated code exists.
-   Test one feature at a time.

# 16. Practice Exercises

Try these without copying a complete solution first.

1.  **Movement:** Change the player's movement speed using the
    Inspector.
2.  **Rotation:** Make a coin rotate continuously.
3.  **Score:** Make each coin give 5 points instead of 1.
4.  **Health:** Create a variable named `health` starting at 100.
5.  **Damage:** Reduce health by 10 when the player touches a hazard.
6.  **Spawner:** Spawn a cube when the player presses Space.
7.  **Timer:** Use a coroutine to wait three seconds before showing a
    message.
8.  **Win condition:** Display a message when the player collects all
    coins.

# 17. Recommended Learning Order

Follow this order to build your Unity scripting foundation:

1.  C# variables, data types, conditions, loops, and methods.
2.  GameObjects, Components, and the Inspector.
3.  Unity lifecycle methods: `Awake`, `Start`, and `Update`.
4.  Transform and Vector3.
5.  Input System and player movement.
6.  Rigidbody, Colliders, and Triggers.
7.  Prefabs, Instantiate, and Destroy.
8.  Coroutines and timers.
9.  UI, health, score, and scene management.
10. Script communication, events, and basic game architecture.

## Final reminder

Do not try to memorize every Unity method. Understand what each
component does, when Unity calls each method, and how scripts
communicate. Build small features, test them, and gradually combine them
into a complete game.
