# Advanced Unity 6 Multiplayer Scripting: FishNet + ASP.NET Core

A practical guide to building a server-authoritative multiplayer RPG foundation.

## Technology stack

- Unity 6 — game client and dedicated game server build
- FishNet — real-time multiplayer networking
- Tugboat — transport option for an initial UDP setup
- ASP.NET Core Web API — accounts, profiles, matchmaking and persistent services
- PostgreSQL — durable player and game records
- Entity Framework Core + Npgsql — database access
- JWT bearer tokens — API authentication

> **Version note:** FishNet APIs and setup can change between releases. Use the documentation matching the FishNet version installed in your project. Examples below demonstrate common FishNet patterns such as `NetworkBehaviour`, `[ServerRpc]`, `[ObserversRpc]`, `[TargetRpc]`, and `SyncVar<T>`. Verify namespaces, constructors, attributes, and callback signatures against your installed version before using them in production.

---

## Table of contents

1. [Architecture](#1-architecture)
2. [Create the Unity project](#2-create-the-unity-project)
3. [Install and configure FishNet](#3-install-and-configure-fishnet)
4. [Networked objects and ownership](#4-networked-objects-and-ownership)
5. [Server-authoritative movement](#5-server-authoritative-movement)
6. [Synchronize health](#6-synchronize-health)
7. [RPC communication](#7-rpc-communication)
8. [Server-authoritative combat](#8-server-authoritative-combat)
9. [Create the ASP.NET Core API](#9-create-the-aspnet-core-api)
10. [PostgreSQL and Entity Framework Core](#10-postgresql-and-entity-framework-core)
11. [Registration and password hashing](#11-registration-and-password-hashing)
12. [JWT authentication](#12-jwt-authentication)
13. [Unity API integration](#13-unity-api-integration)
14. [Authenticate FishNet connections](#14-authenticate-fishnet-connections)
15. [Lobby and matchmaking](#15-lobby-and-matchmaking)
16. [Security and anti-cheat](#16-security-and-anti-cheat)
17. [Dedicated server deployment](#17-dedicated-server-deployment)
18. [Testing and debugging](#18-testing-and-debugging)
19. [Suggested project structure](#19-suggested-project-structure)
20. [Practice milestones](#20-practice-milestones)

---

# 1. Architecture

A multiplayer game commonly needs two server-side systems:

- **Game server:** handles real-time gameplay, movement, combat, enemy AI, spawning, and world state.
- **Backend API:** handles registration, login, player profiles, inventory persistence, matchmaking, and other non-frame-critical operations.

Do not send every movement update through an ASP.NET REST endpoint. HTTP request/response traffic is not a replacement for a real-time multiplayer networking framework.

## 1.1 High-level architecture

```text
                         INTERNET
                            |
             +--------------+--------------+
             |                             |
        HTTPS / REST                    UDP game traffic
             |                             |
             v                             v
    +------------------+          +----------------------+
    | ASP.NET Core API |          | FishNet Game Server  |
    |                  |          |                      |
    | Registration     |          | Player movement      |
    | Login / JWT      |          | Combat validation    |
    | Player profiles  |          | Enemy AI             |
    | Inventory        |          | Match state          |
    | Matchmaking      |          | World synchronization|
    +--------+---------+          +-----------+----------+
             |                                |
             v                                |
    +------------------+                      |
    | PostgreSQL       |<---------------------+
    | Accounts         |    Persist trusted
    | Characters       |    match results
    | Inventory        |
    | Match records    |
    +------------------+
```

## 1.2 Responsibilities

| Feature | Responsible system |
|---|---|
| Registration and login | ASP.NET Core |
| Password hashing | ASP.NET Core |
| Player profile | ASP.NET Core + PostgreSQL |
| Character records | ASP.NET Core + PostgreSQL |
| Player movement | FishNet game server |
| Combat and damage | FishNet game server |
| Enemy AI | FishNet game server |
| Live health synchronization | FishNet |
| Inventory persistence | Backend/database |
| Matchmaking | Backend service |
| Match simulation | FishNet |
| Match result persistence | Trusted game server + backend |

## 1.3 Server authority

The client sends requests such as:

- "I want to move forward."
- "I want to attack this enemy."
- "I want to use this potion."

The server validates the request and applies the result.

The client must not decide authoritative values such as:

- Its own health
- Its own currency
- The amount of damage dealt
- Whether a boss is defeated
- Whether an item is owned

The server is the source of truth for gameplay.

---

# 2. Create the Unity project

Create a Unity 6 3D project named `MultiplayerRPG`.

Suggested folders:

```text
Assets/
    _Project/
        Scripts/
            Networking/
            Player/
            Combat/
            UI/
            Managers/
            Backend/
        Prefabs/
            Player/
            Enemies/
        Scenes/
            MainMenu.unity
            Lobby.unity
            GameWorld.unity
        Materials/
        Models/
```

Install or configure:

- Input System
- TextMeshPro
- FishNet, following its official installation instructions

---

# 3. Install and configure FishNet

FishNet is the networking framework for the Unity project.

Official resources:

- Documentation: https://fish-networking.gitbook.io/docs/
- GitHub: https://github.com/FirstGearGames/FishNet

## 3.1 NetworkManager

A typical FishNet scene uses a NetworkManager with supporting managers/components such as:

- Transport manager
- Server manager
- Client manager
- Time manager
- Observer manager
- Scene manager

Use the FishNet setup wizard or the documentation for your installed version to create and configure these components.

## 3.2 Transport

Tugboat is a common FishNet transport option for an initial UDP-based project.

Keep these concepts separate:

- FishNet provides networking abstractions and object management.
- The transport carries network messages.
- Your `NetworkBehaviour` scripts implement gameplay behavior.

## 3.3 First connection test

Before building gameplay:

1. Create a scene with the FishNet NetworkManager.
2. Configure a transport.
3. Start a server.
4. Connect a client.
5. Confirm that the connection succeeds.
6. Test with two separate game instances.

Do not move forward until basic client/server connectivity works.

---

# 4. Networked objects and ownership

A normal GameObject exists locally. A networked object is managed by FishNet so its identity, lifecycle, and state can be represented across clients.

A networked player prefab generally needs:

- `NetworkObject`
- One or more `NetworkBehaviour` scripts
- Appropriate Collider
- Rigidbody if required by the movement design
- `NetworkTransform` if using transform replication

Register the prefab with FishNet's spawnable prefabs configuration as required by your installed version.

## 4.1 NetworkBehaviour

Use `NetworkBehaviour` instead of `MonoBehaviour` for scripts that need FishNet networking features.

```csharp
using FishNet.Object;
using UnityEngine;

public class NetworkPlayer : NetworkBehaviour
{
    public override void OnStartServer()
    {
        base.OnStartServer();
        Debug.Log("Player started on the server.");
    }

    public override void OnStartClient()
    {
        base.OnStartClient();
        Debug.Log("Player started on a client.");
    }
}
```

| Term | Meaning |
|---|---|
| NetworkObject | Network identity and lifecycle |
| NetworkBehaviour | Base class for network-aware scripts |
| Owner | Client that owns a network object |
| Server | Authority managing the game |
| Observer | Client allowed to observe an object |
| Spawn | Create a networked object |
| Despawn | Remove a networked object |

Common checks include:

```csharp
if (IsServerInitialized)
{
    // Server-side logic.
}

if (IsClientInitialized)
{
    // Client-side logic.
}

if (IsOwner)
{
    // Logic for the owning client.
}
```

`IsOwner` does not mean the client is trusted to decide authoritative gameplay outcomes.

---

# 5. Server-authoritative movement

The intended flow:

1. The owning client reads local input.
2. The client sends an input request to the server.
3. The server validates the request.
4. The server moves the player.
5. A server-authoritative NetworkTransform replicates the result.

## 5.1 Player prefab setup

Create a Capsule named `Player`. Add:

- NetworkObject
- NetworkTransform
- CapsuleCollider
- NetworkPlayerMovement script
- Rigidbody only if required by your movement implementation

Configure NetworkTransform for server authority using the options available in your FishNet version. The server must be the authority publishing the actual transform.

## 5.2 NetworkPlayerMovement.cs

This small example uses the Unity Input System and server-side transform movement. It is for learning the authority flow, not a production controller.

```csharp
using FishNet.Object;
using UnityEngine;
using UnityEngine.InputSystem;

public class NetworkPlayerMovement : NetworkBehaviour
{
    [SerializeField] private float moveSpeed = 5f;

    private void Update()
    {
        // Only the owning client reads its keyboard.
        if (!IsOwner || Keyboard.current == null)
            return;

        Vector2 input = Vector2.zero;

        if (Keyboard.current.wKey.isPressed) input.y += 1;
        if (Keyboard.current.sKey.isPressed) input.y -= 1;
        if (Keyboard.current.aKey.isPressed) input.x -= 1;
        if (Keyboard.current.dKey.isPressed) input.x += 1;

        // Send requested input to the server.
        SubmitMovementServerRpc(input);
    }

    [ServerRpc]
    private void SubmitMovementServerRpc(Vector2 requestedInput)
    {
        // Prevent diagonal input from exceeding magnitude 1.
        Vector2 safeInput =
            Vector2.ClampMagnitude(requestedInput, 1f);

        Vector3 direction = new Vector3(
            safeInput.x,
            0f,
            safeInput.y
        );

        // The server applies the authoritative movement.
        transform.position +=
            direction * moveSpeed * Time.deltaTime;
    }
}
```

### Production improvements

This sample sends an RPC every rendered frame. That is simple to understand but inefficient and unsuitable as a final movement architecture.

For production:

- Send input at a controlled network tick rate.
- Use FishNet's time/tick system.
- Apply movement on the server at a fixed simulation interval.
- Validate input magnitude and movement states.
- Use server-side collision and movement rules.
- Configure NetworkTransform interpolation.
- Add client-side prediction and reconciliation if needed.
- Never trust a client-provided position or velocity.

For physics movement, perform Rigidbody operations on the appropriate physics tick and keep authority consistent.

---

# 6. Synchronize health

FishNet SyncTypes can replicate values such as health, mana, level, team, or character state.

## 6.1 NetworkHealth.cs

```csharp
using FishNet.Object;
using FishNet.Object.Synchronizing;
using UnityEngine;

public class NetworkHealth : NetworkBehaviour
{
    [SerializeField] private int maxHealth = 100;

    // Server writes; observing clients receive synchronized state.
    private readonly SyncVar<int> currentHealth = new();

    public int CurrentHealth => currentHealth.Value;

    public override void OnStartServer()
    {
        base.OnStartServer();
        currentHealth.Value = maxHealth;
    }

    [Server]
    public void TakeDamage(int damage)
    {
        if (damage <= 0 || currentHealth.Value <= 0)
            return;

        currentHealth.Value = Mathf.Clamp(
            currentHealth.Value - damage,
            0,
            maxHealth
        );

        if (currentHealth.Value == 0)
            HandleDeath();
    }

    [Server]
    private void HandleDeath()
    {
        Debug.Log("Player died on the server.");
    }
}
```

> FishNet SyncType constructors and configuration syntax can differ by release. Verify the `SyncVar<T>` declaration and imports against your installed package documentation.

The client should display the synchronized health value. It should not independently decide the actual health.

## 6.2 Simple health UI

```csharp
using UnityEngine;
using TMPro;

public class HealthUI : MonoBehaviour
{
    [SerializeField] private TMP_Text healthText;
    [SerializeField] private NetworkHealth playerHealth;

    private void Update()
    {
        if (playerHealth == null)
            return;

        healthText.text =
            "HP: " + playerHealth.CurrentHealth;
    }
}
```

For a production UI, subscribe to the SyncVar change event rather than polling every frame. Check the event signature in your installed FishNet version.

---

# 7. RPC communication

RPC means Remote Procedure Call. It allows a method to be invoked across the network.

## 7.1 ServerRpc

A client uses a ServerRpc to send a request to the server.

```csharp
using FishNet.Object;
using UnityEngine;

public class NetworkInteraction : NetworkBehaviour
{
    public void RequestInteraction()
    {
        if (!IsOwner)
            return;

        InteractServerRpc();
    }

    [ServerRpc]
    private void InteractServerRpc()
    {
        Debug.Log("Server received an interaction request.");
    }
}
```

For owner-controlled objects, the default ownership requirement is useful. Do not disable it without a deliberate authorization design.

## 7.2 ObserversRpc

An ObserversRpc can notify clients observing the object.

```csharp
using FishNet.Object;
using UnityEngine;

public class NetworkEffects : NetworkBehaviour
{
    [Server]
    public void PlayAttackEffect()
    {
        PlayAttackEffectObserversRpc();
    }

    [ObserversRpc]
    private void PlayAttackEffectObserversRpc()
    {
        Debug.Log("Play local attack effect.");
    }
}
```

Use this for visual effects, sounds, and presentation events.

Do not use an RPC as a replacement for persistent state. A late-joining client needs the current health value, not merely an old damage event.

## 7.3 TargetRpc

A TargetRpc sends a message to a specific client.

```csharp
using FishNet.Object;
using FishNet.Connection;
using UnityEngine;

public class NetworkNotification : NetworkBehaviour
{
    [Server]
    public void NotifyOwner()
    {
        NotifyOwnerTargetRpc(Owner);
    }

    [TargetRpc]
    private void NotifyOwnerTargetRpc(NetworkConnection connection)
    {
        Debug.Log("Private server notification received.");
    }
}
```

TargetRpc is useful for private notifications and owner-specific feedback. Verify the exact parameter convention in your installed FishNet version.

## 7.4 RPC selection

| Requirement | Approach |
|---|---|
| Client requests an action | ServerRpc |
| Server changes shared state | Server logic + SyncTypes |
| Server announces a visual effect | ObserversRpc |
| Server sends private feedback | TargetRpc |
| Persistent player data | Backend/database |
| Current networked health | SyncVar |

---

# 8. Server-authoritative combat

A basic combat flow:

```text
Player presses attack
        |
        v
Client sends attack request
        |
        v
Server checks:
- Player alive?
- Weapon equipped?
- Cooldown finished?
- Target in range?
- Target visible?
        |
        v
Server calculates damage
        |
        v
Server updates target health
        |
        v
FishNet synchronizes health
        |
        v
Observers play attack effects
```

## 8.1 NetworkCombat.cs

This is a compact server-side hit-check example. A production system also needs team rules, line-of-sight checks, weapon configuration, animation timing, and target validation.

```csharp
using FishNet.Object;
using UnityEngine;

public class NetworkCombat : NetworkBehaviour
{
    [SerializeField] private float attackRange = 3f;
    [SerializeField] private int damage = 20;
    [SerializeField] private float attackCooldown = 1f;

    private float nextAttackTime;

    public void RequestAttack()
    {
        if (!IsOwner)
            return;

        AttackServerRpc();
    }

    [ServerRpc]
    private void AttackServerRpc()
    {
        if (Time.time < nextAttackTime)
            return;

        nextAttackTime = Time.time + attackCooldown;

        Collider[] hits = Physics.OverlapSphere(
            transform.position,
            attackRange
        );

        foreach (Collider hit in hits)
        {
            if (hit.gameObject == gameObject)
                continue;

            NetworkHealth target =
                hit.GetComponentInParent<NetworkHealth>();

            if (target == null)
                continue;

            // Add team, line-of-sight, and target eligibility checks.
            target.TakeDamage(damage);
            break;
        }

        AttackEffectObserversRpc();
    }

    [ObserversRpc]
    private void AttackEffectObserversRpc()
    {
        Debug.Log("Play attack animation or effect.");
    }
}
```

The server selects and validates the target. Never let the client submit an arbitrary damage value or declare that an enemy is dead.

---

# 9. Create the ASP.NET Core API

The backend handles operations that must persist beyond a game session:

- Account registration
- Login
- Player profiles
- Character selection
- Inventory records
- Matchmaking queue
- Match history

Create a separate backend project.

## 9.1 Create the Web API

Install a .NET SDK supported by your environment. For a project targeting .NET 10:

```bash
dotnet new webapi -n MultiplayerRPG.Api
cd MultiplayerRPG.Api
```

Add packages:

```bash
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

Use package versions compatible with your target .NET and EF Core versions.

## 9.2 Suggested backend folders

```text
MultiplayerRPG.Api/
    Controllers/
        AuthController.cs
        PlayersController.cs
        CharactersController.cs
        MatchmakingController.cs
    Data/
        GameDbContext.cs
    Models/
        PlayerAccount.cs
        Character.cs
        InventoryItem.cs
        MatchRecord.cs
    DTOs/
        RegisterRequest.cs
        LoginRequest.cs
        AuthResponse.cs
    Services/
        AuthService.cs
        TokenService.cs
        CharacterService.cs
    Program.cs
    appsettings.json
```

Keep controllers responsible for HTTP handling. Put business rules in services.

---

# 10. PostgreSQL and Entity Framework Core

## 10.1 Connection string

Example `appsettings.Development.json`:

```json
{
  "ConnectionStrings": {
    "GameDatabase": "Host=localhost;Port=5432;Database=multiplayer_rpg;Username=postgres;Password=YOUR_LOCAL_PASSWORD"
  }
}
```

Do not commit production passwords or secrets to source control. Use environment variables or a secrets manager for deployment.

## 10.2 PlayerAccount model

`Models/PlayerAccount.cs`:

```csharp
namespace MultiplayerRPG.Api.Models;

public class PlayerAccount
{
    public Guid Id { get; set; }

    public string Username { get; set; } = string.Empty;

    public string Email { get; set; } = string.Empty;

    // Store a password hash, never a plain-text password.
    public string PasswordHash { get; set; } = string.Empty;

    public DateTime CreatedAtUtc { get; set; } =
        DateTime.UtcNow;
}
```

## 10.3 Character model

`Models/Character.cs`:

```csharp
namespace MultiplayerRPG.Api.Models;

public class Character
{
    public Guid Id { get; set; }

    public Guid PlayerAccountId { get; set; }

    public string CharacterName { get; set; } = string.Empty;

    public int Level { get; set; } = 1;

    public long Experience { get; set; }

    public PlayerAccount? PlayerAccount { get; set; }
}
```

## 10.4 DbContext

`Data/GameDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using MultiplayerRPG.Api.Models;

namespace MultiplayerRPG.Api.Data;

public class GameDbContext : DbContext
{
    public GameDbContext(
        DbContextOptions<GameDbContext> options)
        : base(options)
    {
    }

    public DbSet<PlayerAccount> PlayerAccounts =>
        Set<PlayerAccount>();

    public DbSet<Character> Characters =>
        Set<Character>();

    protected override void OnModelCreating(
        ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<PlayerAccount>()
            .HasIndex(x => x.Username)
            .IsUnique();

        modelBuilder.Entity<PlayerAccount>()
            .HasIndex(x => x.Email)
            .IsUnique();

        modelBuilder.Entity<Character>()
            .HasOne(x => x.PlayerAccount)
            .WithMany()
            .HasForeignKey(x => x.PlayerAccountId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

## 10.5 Register PostgreSQL

In `Program.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using MultiplayerRPG.Api.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddDbContext<GameDbContext>(options =>
{
    options.UseNpgsql(
        builder.Configuration.GetConnectionString(
            "GameDatabase"
        )
    );
});

var app = builder.Build();

app.MapControllers();

app.Run();
```

## 10.6 Create migration

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

For production, use a controlled migration process. Do not let every game server instance independently change the database schema during startup.

---

# 11. Registration and password hashing

## 11.1 DTOs

`DTOs/RegisterRequest.cs`:

```csharp
namespace MultiplayerRPG.Api.DTOs;

public record RegisterRequest(
    string Username,
    string Email,
    string Password
);
```

`DTOs/LoginRequest.cs`:

```csharp
namespace MultiplayerRPG.Api.DTOs;

public record LoginRequest(
    string Username,
    string Password
);
```

`DTOs/AuthResponse.cs`:

```csharp
namespace MultiplayerRPG.Api.DTOs;

public record AuthResponse(
    string AccessToken,
    DateTime ExpiresAtUtc
);
```

## 11.2 Use a password hasher

For a straightforward ASP.NET Core application, use the framework's password hasher rather than inventing a hashing algorithm.

Example registration logic:

```csharp
using Microsoft.AspNetCore.Identity;

var passwordHasher =
    new PasswordHasher<PlayerAccount>();

string passwordHash = passwordHasher.HashPassword(
    playerAccount,
    request.Password
);

playerAccount.PasswordHash = passwordHash;
```

Example login verification:

```csharp
var result = passwordHasher.VerifyHashedPassword(
    playerAccount,
    playerAccount.PasswordHash,
    request.Password
);

if (result == PasswordVerificationResult.Failed)
{
    // Reject login.
}
```

A production registration flow also needs input validation, uniqueness checks, rate limiting, safe error responses, password policy, and abuse prevention.

Never return a password or password hash to the client.

---

# 12. JWT authentication

JWT is a signed token representing an authenticated identity.

Useful claims include:

- Subject/account ID
- Username
- Issuer
- Audience
- Expiration

## 12.1 Configuration

Example configuration shape:

```json
{
  "Jwt": {
    "Issuer": "MultiplayerRPG.Api",
    "Audience": "MultiplayerRPG.Client",
    "ExpirationMinutes": 30
  }
}
```

The signing secret must be provided securely through environment configuration or a secret store. Do not commit the secret to this file.

## 12.2 Configure bearer authentication

Simplified `Program.cs` example:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

string jwtSecret =
    builder.Configuration["Jwt:Secret"]
    ?? throw new InvalidOperationException(
        "JWT secret is missing."
    );

builder.Services
    .AddAuthentication(
        JwtBearerDefaults.AuthenticationScheme
    )
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters =
            new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidIssuer =
                    builder.Configuration["Jwt:Issuer"],

                ValidateAudience = true,
                ValidAudience =
                    builder.Configuration["Jwt:Audience"],

                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,

                IssuerSigningKey =
                    new SymmetricSecurityKey(
                        Encoding.UTF8.GetBytes(jwtSecret)
                    ),

                ClockSkew = TimeSpan.FromSeconds(30)
            };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

Use a cryptographically strong key and plan for key rotation. Consider asymmetric signing when several services need to validate tokens without sharing the signing secret.

## 12.3 Protect an endpoint

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

namespace MultiplayerRPG.Api.Controllers;

[ApiController]
[Route("api/players")]
[Authorize]
public class PlayersController : ControllerBase
{
    [HttpGet("me")]
    public IActionResult GetMe()
    {
        string? accountId =
            User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

        return Ok(new { AccountId = accountId });
    }
}
```

A request without a valid token should be rejected.

---

# 13. Unity API integration

Unity sends login credentials to ASP.NET Core over HTTPS. The API returns a JWT access token.

The token is then used for protected API requests and presented to the game server through its connection-authentication flow.

## 13.1 Login flow

```text
Unity Login Screen
        |
        v
POST /api/auth/login
        |
        v
ASP.NET Core validates credentials
        |
        v
API returns signed JWT
        |
        v
Unity receives token
        |
        v
Unity connects to FishNet server
        |
        v
FishNet server authenticates connection
```

## 13.2 Unity request models

```csharp
using System;

[Serializable]
public class LoginRequest
{
    public string username;
    public string password;
}

[Serializable]
public class LoginResponse
{
    public string accessToken;
    public string expiresAtUtc;
}
```

## 13.3 Unity login service

```csharp
using System.Collections;
using System.Text;
using UnityEngine;
using UnityEngine.Networking;

public class ApiLogin : MonoBehaviour
{
    [SerializeField]
    private string apiUrl = "https://localhost:7001";

    public string AccessToken { get; private set; }

    public IEnumerator Login(
        string username,
        string password
    )
    {
        LoginRequest body = new LoginRequest
        {
            username = username,
            password = password
        };

        string json = JsonUtility.ToJson(body);

        using UnityWebRequest request =
            new UnityWebRequest(
                apiUrl + "/api/auth/login",
                "POST"
            );

        byte[] bodyBytes = Encoding.UTF8.GetBytes(json);

        request.uploadHandler =
            new UploadHandlerRaw(bodyBytes);

        request.downloadHandler =
            new DownloadHandlerBuffer();

        request.SetRequestHeader(
            "Content-Type",
            "application/json"
        );

        yield return request.SendWebRequest();

        if (request.result != UnityWebRequest.Result.Success)
        {
            Debug.LogError("Login failed: " + request.error);
            yield break;
        }

        LoginResponse response =
            JsonUtility.FromJson<LoginResponse>(
                request.downloadHandler.text
            );

        if (response == null ||
            string.IsNullOrWhiteSpace(response.accessToken))
        {
            Debug.LogError("Invalid login response.");
            yield break;
        }

        AccessToken = response.accessToken;
        Debug.Log("Login successful.");
    }
}
```

This is a learning example. A real project should also validate expiration, avoid logging tokens, store credentials securely, implement refresh or reauthentication, and handle API errors.

## 13.4 Send a bearer token

For a protected API request:

```csharp
request.SetRequestHeader(
    "Authorization",
    "Bearer " + AccessToken
);
```

Do not put long-lived account credentials into game server logs or public network messages.

---

# 14. Authenticate FishNet connections

**Important:** Receiving a JWT from ASP.NET Core does not automatically authenticate a FishNet connection.

The FishNet server must independently validate the credential presented during its connection/authentication workflow.

A secure integration should:

1. Use FishNet's authentication workflow for the installed version.
2. Receive the token through the intended authentication payload.
3. Validate the signature and claims on the game server.
4. Check expiration, issuer, audience, and account status.
5. Bind the authenticated account ID to the connection.
6. Reject invalid or expired credentials.
7. Avoid trusting an account ID sent separately by the client.

Use the official FishNet authenticator documentation for the exact authenticator interfaces, payload types, and connection callbacks in your installed version.

For a larger architecture, consider asymmetric JWT signing or a short-lived, purpose-specific game-session ticket. A game-session ticket can reduce the exposure of the general API access token.

---

# 15. Lobby and matchmaking

Matchmaking selects a game server or creates a match for players.

```text
Unity Client
    |
    v
POST /api/matchmaking/queue
    |
    v
ASP.NET Core
    |
    v
Find compatible players
    |
    v
Allocate game server
    |
    v
Return session information
    |
    v
Unity connects to FishNet server
```

## 15.1 Match record

```csharp
public class MatchRecord
{
    public Guid Id { get; set; }

    public string Status { get; set; } = "Waiting";

    public DateTime CreatedAtUtc { get; set; } =
        DateTime.UtcNow;

    public DateTime? StartedAtUtc { get; set; }

    public DateTime? FinishedAtUtc { get; set; }
}
```

Use controlled status values in a real application, such as:

- Waiting
- Allocating
- Ready
- InProgress
- Completed
- Cancelled
- Failed

## 15.2 Prevent duplicate matchmaking

Two API requests from one account can arrive nearly simultaneously.

Use database constraints and transactions to prevent one player from entering multiple active queues or matches unintentionally.

Possible approaches:

- Unique active queue entry per player.
- Transactional match allocation.
- Distributed locks for multiple matchmaking workers.
- Idempotency keys for retryable requests.

Do not rely only on disabling a Unity button.

## 15.3 Game server allocation

A small project can use a fixed game server address. A larger project may need an allocator that can:

- Find an available server.
- Start a dedicated server process or container.
- Assign a port.
- Track server capacity.
- Return connection details.
- Shut down idle servers.

---

# 16. Security and anti-cheat

A multiplayer game is exposed to modified clients, automated requests, packet manipulation, and replay attempts.

Security must be part of the architecture.

## 16.1 Never trust client values

Unsafe:

```csharp
[ServerRpc]
private void SetHealthServerRpc(int health)
{
    currentHealth.Value = health;
}
```

A client could request an arbitrary health value.

Better:

```csharp
[Server]
public void ApplyDamage(int damage)
{
    if (damage <= 0)
        return;

    currentHealth.Value = Mathf.Max(
        0,
        currentHealth.Value - damage
    );
}
```

The server calculates and applies changes according to game rules.

## 16.2 Validate movement

Check:

- Input magnitude
- Maximum movement speed
- Allowed movement states
- Stun and root effects
- Collision constraints
- Teleport permissions
- Movement timing
- Cooldowns

## 16.3 Validate combat

Check:

- Weapon equipped
- Attack cooldown
- Target range
- Line of sight
- Team restrictions
- Target alive
- Weapon damage configuration
- Skill requirements
- Resource costs

## 16.4 Validate inventory

The client must not decide item quantities or currency.

Validate:

- Item ownership
- Quantity
- Purchase cost
- Currency balance
- Trade rules
- Item-use restrictions

Use database transactions for operations involving multiple persistent records.

## 16.5 Rate limiting

Apply limits to sensitive endpoints:

- Login attempts
- Registration
- Matchmaking requests
- Inventory actions
- Reward claims

Also validate network message frequency and payload size on the game server.

---

# 17. Dedicated server deployment

A multiplayer game usually has at least two deployable services:

1. ASP.NET Core backend.
2. FishNet dedicated game server.

PostgreSQL is a separate data service.

## 17.1 Development environment

```text
Developer PC
    |
    +-- Unity Editor
    +-- ASP.NET Core API
    +-- PostgreSQL
    +-- FishNet server build
```

## 17.2 Production environment

```text
                  INTERNET
                     |
           +---------+---------+
           |                   |
         HTTPS                 UDP
           |                   |
           v                   v
    +-------------+     +----------------+
    | API Hosting |     | Game Server    |
    | ASP.NET     |     | FishNet        |
    +------+------+     +-------+--------+
           |                    |
           v                    |
    +-------------+             |
    | PostgreSQL  |<------------+
    +-------------+
```

## 17.3 Dedicated server build

Unity supports dedicated server builds. Configure the Dedicated Server build target and test the actual build, not only Play Mode in the Editor.

A dedicated server should:

- Run without rendering where possible.
- Load only required scenes and assets.
- Accept network connections.
- Validate player actions.
- Save match results.
- Shut down cleanly.

## 17.4 Network ports

Configure:

- HTTPS access for the API.
- The selected FishNet transport's UDP port.
- Firewall rules.
- Hosting provider security rules.

The actual UDP port depends on your transport configuration. Do not assume a port number from an unrelated tutorial.

## 17.5 Production checklist

- [ ] HTTPS certificate configured.
- [ ] JWT signing secret protected.
- [ ] PostgreSQL credentials protected.
- [ ] Database backups configured.
- [ ] Game server firewall configured.
- [ ] Dedicated server build tested.
- [ ] Server logs configured.
- [ ] API rate limiting enabled.
- [ ] Server authentication enabled.
- [ ] Match results validated server-side.
- [ ] Error monitoring configured.
- [ ] Database migrations managed safely.

---

# 18. Testing and debugging

Test with separate clients. Do not rely exclusively on one Unity Editor instance.

## 18.1 Test cases

| Test | Expected result |
|---|---|
| One client connects | Server accepts valid client |
| Two clients connect | Both players appear |
| Client moves | Other client sees movement |
| Client attacks | Server validates attack |
| Player takes damage | Health synchronizes |
| Player disconnects | Server handles disconnect |
| Invalid JWT | Connection rejected |
| Expired JWT | Connection rejected |
| Excessive movement | Server clamps or rejects it |
| Invalid damage request | Server ignores/rejects it |
| Match completes | Trusted result is saved |

## 18.2 Debugging checklist

When a feature fails, identify which side owns the problem:

1. Is the code running on the client?
2. Is the code running on the server?
3. Does the object have a NetworkObject?
4. Is the object spawned?
5. Is the client the owner?
6. Is the client an observer?
7. Is the RPC being called?
8. Is the server accepting the request?
9. Is the synchronized value changing?
10. Is the receiving client displaying the value?

Use clear logs:

```csharp
Debug.Log("[CLIENT] Input received.");
Debug.Log("[SERVER] Attack request received.");
Debug.Log("[SERVER] Damage applied.");
Debug.Log("[CLIENT] Health updated.");
```

Never log access tokens, passwords, signing secrets, or sensitive player data.

---

# 19. Suggested project structure

```text
MultiplayerRPG/
|
+-- UnityClient/
|   +-- Assets/
|       +-- _Project/
|           +-- Scripts/
|               +-- Networking/
|               |   +-- NetworkConnectionManager.cs
|               |   +-- NetworkPlayerMovement.cs
|               |   +-- NetworkHealth.cs
|               +-- Player/
|               |   +-- PlayerController.cs
|               |   +-- PlayerInputReader.cs
|               +-- Combat/
|               |   +-- NetworkCombat.cs
|               |   +-- WeaponData.cs
|               +-- UI/
|               |   +-- LoginUI.cs
|               |   +-- HealthUI.cs
|               +-- Backend/
|                   +-- ApiClient.cs
|                   +-- AuthService.cs
|
+-- GameServer/
|   +-- Unity dedicated server build
|   +-- Server configuration
|   +-- Logs
|
+-- Backend/
    +-- MultiplayerRPG.Api/
        +-- Controllers/
        +-- Data/
        +-- Models/
        +-- DTOs/
        +-- Services/
        +-- Program.cs
```

The game server is built from the Unity project with server-specific configuration. It is not a separate ASP.NET Core gameplay simulation.

---

# 20. Practice milestones

Build the project in stages. Do not implement everything at once.

## Milestone 1 — FishNet connection

- [ ] Install FishNet.
- [ ] Configure NetworkManager.
- [ ] Configure transport.
- [ ] Start a local server.
- [ ] Connect two clients.

**Success condition:** Both clients connect to the same server.

## Milestone 2 — Player spawning

- [ ] Create a player prefab.
- [ ] Add NetworkObject.
- [ ] Register the prefab.
- [ ] Configure player spawning.
- [ ] Confirm ownership.

**Success condition:** Each connected client receives a player object.

## Milestone 3 — Server-authoritative movement

- [ ] Read input using the Input System.
- [ ] Send input requests to the server.
- [ ] Validate input.
- [ ] Move the player on the server.
- [ ] Synchronize transforms.

**Success condition:** Two clients see consistent player movement.

## Milestone 4 — Health and combat

- [ ] Create NetworkHealth.
- [ ] Synchronize health.
- [ ] Create an attack request.
- [ ] Validate range and cooldown.
- [ ] Apply damage on the server.
- [ ] Display health in the UI.

**Success condition:** One player can damage another, and both clients see the correct result.

## Milestone 5 — Backend and database

- [ ] Create Web API.
- [ ] Connect PostgreSQL.
- [ ] Create PlayerAccount.
- [ ] Create registration endpoint.
- [ ] Create login endpoint.
- [ ] Hash passwords.
- [ ] Issue JWT.

**Success condition:** A player can register and log in.

## Milestone 6 — Unity integration

- [ ] Send login requests.
- [ ] Receive JWT.
- [ ] Call a protected profile endpoint.
- [ ] Handle expired tokens.
- [ ] Add FishNet connection authentication.

**Success condition:** Only authenticated players can enter the game server.

## Milestone 7 — Matchmaking

- [ ] Create a matchmaking queue.
- [ ] Prevent duplicate queue entries.
- [ ] Allocate a game server.
- [ ] Return server connection details.
- [ ] Connect players to the assigned server.

**Success condition:** Players can join an assigned multiplayer match.

## Milestone 8 — Deployment

- [ ] Build dedicated server.
- [ ] Deploy ASP.NET Core.
- [ ] Deploy PostgreSQL.
- [ ] Configure HTTPS.
- [ ] Configure UDP networking.
- [ ] Test external connections.
- [ ] Monitor server performance.

**Success condition:** Two or more players can join a hosted game outside the development computer.

---

# Official documentation

Use official documentation as the source of truth when APIs change.

## Unity

- Unity 6 Manual: https://docs.unity3d.com/6000.0/Documentation/Manual/
- Unity 6 Scripting API: https://docs.unity3d.com/6000.0/Documentation/ScriptReference/

## FishNet

- FishNet documentation: https://fish-networking.gitbook.io/docs/
- FishNet GitHub: https://github.com/FirstGearGames/FishNet

## ASP.NET Core

- ASP.NET Core documentation: https://learn.microsoft.com/aspnet/core/
- Web API controllers: https://learn.microsoft.com/aspnet/core/web-api/
- JWT bearer authentication: https://learn.microsoft.com/aspnet/core/security/authentication/

## Entity Framework Core and PostgreSQL

- EF Core: https://learn.microsoft.com/ef/core/
- Npgsql EF Core provider: https://www.npgsql.org/efcore/

---

# Final advice

The most important design decision is separating **real-time gameplay authority** from **persistent backend services**.

FishNet should manage the live multiplayer world. ASP.NET Core should manage accounts, persistent records, and matchmaking services. PostgreSQL should store durable data.

Start with two clients moving around one small scene. Add health and combat next. Only after those systems work should you introduce account authentication, matchmaking, and dedicated server deployment.

A working two-player prototype is a better foundation than a large multiplayer architecture that has never been tested.
