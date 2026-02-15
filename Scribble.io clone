# 📚 Step-by-Step Tutorial: Building a Real-Time Multiplayer Drawing Game

## Complete Guide from Zero to Deployment

This tutorial will guide you through building a complete real-time multiplayer drawing game (like Skribbl.io) from scratch. You'll learn SignalR, WebSockets, Canvas API, and modern web development practices.

---

## 📋 Table of Contents

1. [Prerequisites & Setup](#step-1-prerequisites--setup)
2. [Understanding the Architecture](#step-2-understanding-the-architecture)
3. [Backend - Project Setup](#step-3-backend---project-setup)
4. [Backend - Creating Models](#step-4-backend---creating-models)
5. [Backend - Game Manager Service](#step-5-backend---game-manager-service)
6. [Backend - SignalR Hub](#step-6-backend---signalr-hub)
7. [Backend - Configuration](#step-7-backend---configuration)
8. [Frontend - Project Setup](#step-8-frontend---project-setup)
9. [Frontend - SignalR Service](#step-9-frontend---signalr-service)
10. [Frontend - Lobby Component](#step-10-frontend---lobby-component)
11. [Frontend - Game Component](#step-11-frontend---game-component)
12. [Frontend - Canvas Drawing](#step-12-frontend---canvas-drawing)
13. [Frontend - Styling](#step-13-frontend---styling)
14. [Testing the Application](#step-14-testing-the-application)
15. [Deployment](#step-15-deployment)

---

## Step 1: Prerequisites & Setup

### What You Need to Install

**1. Install .NET 8.0 SDK**
```bash
# Download from: https://dotnet.microsoft.com/download/dotnet/8.0
# Verify installation
dotnet --version
# Should show: 8.0.x
```

**Why?** .NET is Microsoft's framework for building web APIs and services.

**2. Install Node.js (18 or higher)**
```bash
# Download from: https://nodejs.org/
# Verify installation
node --version
# Should show: v18.x or higher

npm --version
# Should show: 9.x or higher
```

**Why?** Node.js runs the Angular development server and build tools.

**3. Install Visual Studio Code**
```bash
# Download from: https://code.visualstudio.com/
```

**Why?** Best code editor with excellent extensions for .NET and Angular.

**4. Install VS Code Extensions**
- C# (Microsoft)
- Angular Language Service
- ESLint
- Prettier

### Create Project Folder

```bash
# Create main project directory
mkdir scribble-game
cd scribble-game

# Create backend and frontend folders
mkdir backend
mkdir frontend
```

**Project Structure Preview:**
```
scribble-game/
├── backend/         # ASP.NET Core API
└── frontend/        # Angular app
```

---

## Step 2: Understanding the Architecture

### How Real-Time Games Work

**Traditional Web (HTTP):**
```
Client: "Hey server, any updates?"
Server: "Nope"
Client: "How about now?"
Server: "Nope"
(Repeats every second - inefficient!)
```

**WebSocket (SignalR):**
```
Client: Connects once
Server: "Player joined!"
Client: Receives instantly
Server: "New drawing data!"
Client: Updates immediately
(Persistent connection - efficient!)
```

### Application Flow

```
1. User opens app
   ↓
2. Angular app loads
   ↓
3. Connects to SignalR Hub (WebSocket)
   ↓
4. User creates/joins room
   ↓
5. Server adds user to group
   ↓
6. User draws on canvas
   ↓
7. Coordinates sent via SignalR
   ↓
8. Server broadcasts to other players
   ↓
9. Other players' canvases update
   ↓
10. Repeat in real-time!
```

### Key Technologies Explained

**SignalR:**
- Real-time communication library
- Uses WebSockets (fast!)
- Falls back to other methods if WebSockets unavailable
- Handles connection management automatically

**Angular:**
- Frontend framework by Google
- Component-based architecture
- TypeScript for type safety
- Reactive programming with RxJS

**Canvas API:**
- HTML5 drawing surface
- Pixel-based graphics
- Fast rendering
- Good for games and interactive graphics

---

## Step 3: Backend - Project Setup

### Create the ASP.NET Core Project

```bash
cd backend

# Create new Web API project
dotnet new webapi -n ScribbleGame.API

# Navigate into project
cd ScribbleGame.API

# Install SignalR (already included in ASP.NET Core, but we'll be explicit)
dotnet add package Microsoft.AspNetCore.SignalR

# Restore dependencies
dotnet restore
```

**What Just Happened?**
- Created a new Web API project
- Configured for REST and SignalR
- Installed all necessary packages

### Verify It Works

```bash
# Run the project
dotnet run

# You should see:
# Now listening on: https://localhost:5001
# Now listening on: http://localhost:5000
```

**Test it:** Open browser to `https://localhost:5001` - you should see a Swagger API documentation page.

Press `Ctrl+C` to stop the server.

---

## Step 4: Backend - Creating Models

### Understanding Models

Models are data structures that represent objects in our game. Think of them as blueprints.

### Create the Player Model

**Why?** We need to track player information: name, score, connection status.

Create file: `Models/Player.cs`

```csharp
namespace ScribbleGame.API.Models
{
    /// <summary>
    /// Represents a player in the game
    /// </summary>
    public class Player
    {
        // SignalR connection ID - unique identifier for each connection
        public string ConnectionId { get; set; } = string.Empty;
        
        // Player's chosen username
        public string Username { get; set; } = string.Empty;
        
        // Current score (accumulates across rounds)
        public int Score { get; set; }
        
        // Is this player currently drawing?
        public bool IsDrawing { get; set; }
        
        // Has this player guessed correctly this round?
        public bool HasGuessedCorrectly { get; set; }
        
        // When did the player join?
        public DateTime JoinedAt { get; set; } = DateTime.UtcNow;
    }
}
```

**Line-by-Line Explanation:**
- `ConnectionId`: SignalR assigns each connection a unique ID
- `Username`: Display name shown to other players
- `Score`: Running total of points
- `IsDrawing`: Only one player draws at a time
- `HasGuessedCorrectly`: Prevents guessing multiple times per round
- `JoinedAt`: Useful for ordering players

### Create the GameRoom Model

**Why?** We need to track the entire game state: players, current word, round info.

Create file: `Models/GameRoom.cs`

```csharp
namespace ScribbleGame.API.Models
{
    /// <summary>
    /// Represents a game room/session
    /// </summary>
    public class GameRoom
    {
        // 6-character unique code (e.g., "ABC123")
        public string RoomCode { get; set; } = string.Empty;
        
        // All players in this room
        public List<Player> Players { get; set; } = new();
        
        // ConnectionId of the current drawer
        public string CurrentDrawerId { get; set; } = string.Empty;
        
        // The word being drawn (secret!)
        public string CurrentWord { get; set; } = string.Empty;
        
        // When did this round start?
        public DateTime RoundStartTime { get; set; }
        
        // Which round are we on?
        public int RoundNumber { get; set; }
        
        // How long is each round? (default 80 seconds)
        public int RoundDurationSeconds { get; set; } = 80;
        
        // Current game state
        public GameState State { get; set; } = GameState.Waiting;
        
        // Pool of words to choose from
        public List<string> WordPool { get; set; } = new()
        {
            "elephant", "guitar", "pizza", "rainbow", "computer",
            "mountain", "bicycle", "sunset", "robot", "castle",
            "butterfly", "lighthouse", "dinosaur", "volcano", "spaceship",
            "waterfall", "penguin", "keyboard", "umbrella", "telescope"
        };
        
        // Chat message history
        public List<ChatMessage> ChatHistory { get; set; } = new();
        
        // When was this room created?
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    }

    /// <summary>
    /// Game state enum
    /// </summary>
    public enum GameState
    {
        Waiting,      // Waiting for players to join
        ChoosingWord, // (Future: Let drawer choose from 3 words)
        Drawing,      // Active drawing round
        RoundEnd,     // Round ended, showing results
        GameEnd       // (Future: Game completely over)
    }

    /// <summary>
    /// Chat message model
    /// </summary>
    public class ChatMessage
    {
        public string Username { get; set; } = string.Empty;
        public string Message { get; set; } = string.Empty;
        public DateTime Timestamp { get; set; } = DateTime.UtcNow;
        public bool IsSystemMessage { get; set; }
        public bool IsCorrectGuess { get; set; }
    }
}
```

**Key Concepts:**
- **RoomCode**: Like a room number in a hotel - unique identifier
- **WordPool**: Pre-defined words to draw (easy to expand!)
- **GameState**: Tracks what's happening in the game
- **ChatHistory**: Stores all messages (useful for late joiners)

### Create the DrawingData Model

**Why?** We need to send drawing coordinates between players.

Create file: `Models/DrawingData.cs`

```csharp
namespace ScribbleGame.API.Models
{
    /// <summary>
    /// Represents a drawing stroke/action
    /// </summary>
    public class DrawingData
    {
        // Current X position
        public double X { get; set; }
        
        // Current Y position
        public double Y { get; set; }
        
        // Previous X position (to draw lines)
        public double PrevX { get; set; }
        
        // Previous Y position
        public double PrevY { get; set; }
        
        // Color in hex format (e.g., "#FF0000" for red)
        public string Color { get; set; } = "#000000";
        
        // Brush size in pixels
        public int LineWidth { get; set; } = 2;
        
        // What action? "draw" or "clear"
        public string Action { get; set; } = "draw";
    }
}
```

**Why X, Y, PrevX, PrevY?**
- Drawing is really just connecting dots
- We draw a line from (PrevX, PrevY) to (X, Y)
- This creates smooth, continuous lines

---

## Step 5: Backend - Game Manager Service

### What is a Service?

A service contains business logic - the "brains" of your application. It's separate from the API layer for better organization.

### Why Game Manager?

The Game Manager:
- Stores all active game rooms
- Creates and manages rooms
- Handles player join/leave
- Manages game flow
- Validates guesses
- Calculates scores

### Create the Game Manager

Create file: `Services/GameManager.cs`

```csharp
using ScribbleGame.API.Models;
using System.Collections.Concurrent;

namespace ScribbleGame.API.Services
{
    /// <summary>
    /// Manages all game rooms and game logic
    /// </summary>
    public class GameManager
    {
        // Thread-safe dictionary to store rooms
        // Key: RoomCode, Value: GameRoom
        private readonly ConcurrentDictionary<string, GameRoom> _rooms = new();
        
        // Random number generator for word selection
        private readonly Random _random = new();

        /// <summary>
        /// Create a new game room
        /// </summary>
        public GameRoom CreateRoom(string roomCode)
        {
            var room = new GameRoom
            {
                RoomCode = roomCode.ToUpper(), // Always uppercase for consistency
                State = GameState.Waiting
            };

            // Add room to dictionary
            _rooms.TryAdd(room.RoomCode, room);
            
            return room;
        }

        /// <summary>
        /// Get a room by its code
        /// </summary>
        public GameRoom? GetRoom(string roomCode)
        {
            _rooms.TryGetValue(roomCode.ToUpper(), out var room);
            return room;
        }

        /// <summary>
        /// Remove a room (when empty)
        /// </summary>
        public bool RemoveRoom(string roomCode)
        {
            return _rooms.TryRemove(roomCode.ToUpper(), out _);
        }

        /// <summary>
        /// Add a player to a room
        /// </summary>
        public Player? AddPlayer(string roomCode, string connectionId, string username)
        {
            var room = GetRoom(roomCode);
            if (room == null) return null;

            // Create new player
            var player = new Player
            {
                ConnectionId = connectionId,
                Username = username,
                Score = 0,
                IsDrawing = false
            };

            // Add to room's player list
            room.Players.Add(player);
            
            return player;
        }

        /// <summary>
        /// Remove a player from a room
        /// </summary>
        public bool RemovePlayer(string roomCode, string connectionId)
        {
            var room = GetRoom(roomCode);
            if (room == null) return false;

            // Find the player
            var player = room.Players.FirstOrDefault(p => p.ConnectionId == connectionId);
            if (player == null) return false;

            // Remove from list
            room.Players.Remove(player);

            // If room is empty, delete it
            if (room.Players.Count == 0)
            {
                RemoveRoom(roomCode);
            }

            return true;
        }

        /// <summary>
        /// Start a new round
        /// </summary>
        public void StartNewRound(string roomCode)
        {
            var room = GetRoom(roomCode);
            if (room == null) return;

            // Reset all players' round-specific states
            foreach (var player in room.Players)
            {
                player.HasGuessedCorrectly = false;
                player.IsDrawing = false;
            }

            // Select next drawer (rotate through players)
            var currentDrawerIndex = room.Players.FindIndex(p => p.ConnectionId == room.CurrentDrawerId);
            var nextDrawerIndex = (currentDrawerIndex + 1) % room.Players.Count;
            var nextDrawer = room.Players[nextDrawerIndex];

            // Assign drawer
            room.CurrentDrawerId = nextDrawer.ConnectionId;
            nextDrawer.IsDrawing = true;

            // Pick a random word
            room.CurrentWord = room.WordPool[_random.Next(room.WordPool.Count)];

            // Update round info
            room.RoundStartTime = DateTime.UtcNow;
            room.RoundNumber++;
            room.State = GameState.Drawing;
        }

        /// <summary>
        /// Check if a guess is correct
        /// </summary>
        public bool CheckGuess(string roomCode, string connectionId, string guess)
        {
            var room = GetRoom(roomCode);
            if (room == null) return false;

            var player = room.Players.FirstOrDefault(p => p.ConnectionId == connectionId);
            
            // Can't guess if:
            // - Player not found
            // - Already guessed correctly
            // - Is the drawer
            if (player == null || player.HasGuessedCorrectly || player.IsDrawing) 
                return false;

            // Check if guess matches word (case-insensitive)
            if (string.Equals(guess.Trim(), room.CurrentWord, StringComparison.OrdinalIgnoreCase))
            {
                player.HasGuessedCorrectly = true;

                // Calculate score with time bonus
                var elapsedSeconds = (DateTime.UtcNow - room.RoundStartTime).TotalSeconds;
                var timeBonus = Math.Max(0, room.RoundDurationSeconds - (int)elapsedSeconds);
                
                // Base score + time bonus
                player.Score += 100 + timeBonus;

                return true;
            }

            return false;
        }

        /// <summary>
        /// Get word with letters masked (e.g., "elephant" → "________")
        /// </summary>
        public string GetMaskedWord(string roomCode)
        {
            var room = GetRoom(roomCode);
            if (room == null) return "";

            // Replace each character with underscore
            return new string('_', room.CurrentWord.Length);
        }

        /// <summary>
        /// Get all active rooms
        /// </summary>
        public List<GameRoom> GetAllRooms()
        {
            return _rooms.Values.ToList();
        }

        /// <summary>
        /// Generate a random room code
        /// </summary>
        public string GenerateRoomCode()
        {
            const string chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
            var code = new char[6];

            for (int i = 0; i < 6; i++)
            {
                code[i] = chars[_random.Next(chars.Length)];
            }

            return new string(code);
        }
    }
}
```

**Key Method Explanations:**

1. **CreateRoom**: Creates a new game room with a unique code
2. **AddPlayer**: Adds a player to a room (joining)
3. **RemovePlayer**: Removes player when they disconnect
4. **StartNewRound**: Rotates drawer, picks word, resets states
5. **CheckGuess**: Validates guesses and awards points
6. **GenerateRoomCode**: Creates random 6-character codes

**ConcurrentDictionary?**
- Thread-safe (multiple users can access simultaneously)
- Prevents race conditions
- Essential for web applications

---

## Step 6: Backend - SignalR Hub

### What is a SignalR Hub?

A Hub is like a telephone switchboard:
- Receives calls (method invocations) from clients
- Routes messages to appropriate recipients
- Manages connections

### Create the GameHub

Create file: `Hubs/GameHub.cs`

```csharp
using Microsoft.AspNetCore.SignalR;
using ScribbleGame.API.Models;
using ScribbleGame.API.Services;

namespace ScribbleGame.API.Hubs
{
    /// <summary>
    /// SignalR Hub for real-time game communication
    /// </summary>
    public class GameHub : Hub
    {
        private readonly GameManager _gameManager;
        private readonly ILogger<GameHub> _logger;

        // Dependency injection - framework provides these automatically
        public GameHub(GameManager gameManager, ILogger<GameHub> logger)
        {
            _gameManager = gameManager;
            _logger = logger;
        }

        /// <summary>
        /// Client calls this to create a new room
        /// </summary>
        public async Task CreateRoom(string username)
        {
            // Generate unique room code
            var roomCode = _gameManager.GenerateRoomCode();
            
            // Create the room
            var room = _gameManager.CreateRoom(roomCode);

            // Add the creator as first player
            var player = _gameManager.AddPlayer(
                roomCode, 
                Context.ConnectionId,  // SignalR provides this
                username
            );

            if (player != null)
            {
                // Add this connection to SignalR group
                // Groups allow broadcasting to specific rooms
                await Groups.AddToGroupAsync(Context.ConnectionId, roomCode);

                // Send success response to caller only
                await Clients.Caller.SendAsync("RoomCreated", new
                {
                    roomCode = room.RoomCode,
                    players = room.Players
                });

                _logger.LogInformation($"Room {roomCode} created by {username}");
            }
        }

        /// <summary>
        /// Client calls this to join an existing room
        /// </summary>
        public async Task JoinRoom(string roomCode, string username)
        {
            var room = _gameManager.GetRoom(roomCode);

            // Validation
            if (room == null)
            {
                await Clients.Caller.SendAsync("Error", "Room not found");
                return;
            }

            // Check for duplicate username
            if (room.Players.Any(p => p.Username.Equals(username, StringComparison.OrdinalIgnoreCase)))
            {
                await Clients.Caller.SendAsync("Error", "Username already taken in this room");
                return;
            }

            // Add player
            var player = _gameManager.AddPlayer(roomCode, Context.ConnectionId, username);

            if (player != null)
            {
                // Add to SignalR group
                await Groups.AddToGroupAsync(Context.ConnectionId, roomCode);

                // Notify ALL players in room (including new player)
                await Clients.Group(roomCode).SendAsync("PlayerJoined", new
                {
                    players = room.Players,
                    newPlayer = player
                });

                // Add system message
                var systemMessage = new ChatMessage
                {
                    Username = "System",
                    Message = $"{username} joined the game",
                    IsSystemMessage = true
                };
                room.ChatHistory.Add(systemMessage);

                // Broadcast system message
                await Clients.Group(roomCode).SendAsync("ReceiveMessage", systemMessage);

                _logger.LogInformation($"{username} joined room {roomCode}");
            }
        }

        /// <summary>
        /// Start the game (requires at least 2 players)
        /// </summary>
        public async Task StartGame(string roomCode)
        {
            var room = _gameManager.GetRoom(roomCode);
            if (room == null) return;

            // Need at least 2 players
            if (room.Players.Count < 2)
            {
                await Clients.Caller.SendAsync("Error", "Need at least 2 players to start");
                return;
            }

            // Start first round
            _gameManager.StartNewRound(roomCode);

            var drawer = room.Players.First(p => p.IsDrawing);

            // Tell the drawer the actual word
            await Clients.Client(drawer.ConnectionId).SendAsync("YourTurnToDraw", new
            {
                word = room.CurrentWord,
                roundDuration = room.RoundDurationSeconds
            });

            // Tell everyone else the masked word
            var maskedWord = _gameManager.GetMaskedWord(roomCode);
            await Clients.GroupExcept(roomCode, drawer.ConnectionId).SendAsync("RoundStarted", new
            {
                drawer = drawer.Username,
                wordLength = room.CurrentWord.Length,
                maskedWord = maskedWord,
                roundDuration = room.RoundDurationSeconds
            });

            _logger.LogInformation($"Game started in room {roomCode}");
        }

        /// <summary>
        /// Broadcast drawing data to other players
        /// </summary>
        public async Task SendDrawing(string roomCode, DrawingData drawingData)
        {
            var room = _gameManager.GetRoom(roomCode);
            if (room == null) return;

            // Verify sender is the current drawer
            var player = room.Players.FirstOrDefault(p => p.ConnectionId == Context.ConnectionId);
            if (player == null || !player.IsDrawing) return;

            // Broadcast to everyone EXCEPT the sender
            // (sender already has it on their own canvas)
            await Clients.OthersInGroup(roomCode).SendAsync("ReceiveDrawing", drawingData);
        }

        /// <summary>
        /// Handle chat messages and guesses
        /// </summary>
        public async Task SendMessage(string roomCode, string message)
        {
            var room = _gameManager.GetRoom(roomCode);
            if (room == null) return;

            var player = room.Players.FirstOrDefault(p => p.ConnectionId == Context.ConnectionId);
            if (player == null) return;

            // If sender is drawer, just send as chat (can't guess)
            if (player.IsDrawing)
            {
                var chatMsg = new ChatMessage
                {
                    Username = player.Username,
                    Message = message
                };
                room.ChatHistory.Add(chatMsg);
                await Clients.Group(roomCode).SendAsync("ReceiveMessage", chatMsg);
                return;
            }

            // Check if it's a correct guess
            var isCorrect = _gameManager.CheckGuess(roomCode, Context.ConnectionId, message);

            if (isCorrect)
            {
                // Correct guess!
                var correctMsg = new ChatMessage
                {
                    Username = "System",
                    Message = $"{player.Username} guessed the word!",
                    IsSystemMessage = true,
                    IsCorrectGuess = true
                };
                room.ChatHistory.Add(correctMsg);

                // Notify everyone
                await Clients.Group(roomCode).SendAsync("CorrectGuess", new
                {
                    username = player.Username,
                    score = player.Score
                });

                await Clients.Group(roomCode).SendAsync("ReceiveMessage", correctMsg);

                // Update player scores
                await Clients.Group(roomCode).SendAsync("PlayersUpdated", room.Players);

                // Check if all non-drawers have guessed
                var allGuessed = room.Players
                    .Where(p => !p.IsDrawing)
                    .All(p => p.HasGuessedCorrectly);
                    
                if (allGuessed)
                {
                    await EndRound(roomCode);
                }
            }
            else
            {
                // Regular chat message
                var chatMsg = new ChatMessage
                {
                    Username = player.Username,
                    Message = message
                };
                room.ChatHistory.Add(chatMsg);
                await Clients.Group(roomCode).SendAsync("ReceiveMessage", chatMsg);
            }
        }

        /// <summary>
        /// End the current round
        /// </summary>
        public async Task EndRound(string roomCode)
        {
            var room = _gameManager.GetRoom(roomCode);
            if (room == null) return;

            room.State = GameState.RoundEnd;

            // Show results to everyone
            await Clients.Group(roomCode).SendAsync("RoundEnded", new
            {
                word = room.CurrentWord,
                players = room.Players.OrderByDescending(p => p.Score).ToList()
            });

            _logger.LogInformation($"Round ended in room {roomCode}");
        }

        /// <summary>
        /// Start next round
        /// </summary>
        public async Task NextRound(string roomCode)
        {
            var room = _gameManager.GetRoom(roomCode);
            if (room == null) return;

            // Clear everyone's canvas
            await Clients.Group(roomCode).SendAsync("ClearCanvas");

            // Start new round
            _gameManager.StartNewRound(roomCode);

            var drawer = room.Players.First(p => p.IsDrawing);

            // Notify drawer
            await Clients.Client(drawer.ConnectionId).SendAsync("YourTurnToDraw", new
            {
                word = room.CurrentWord,
                roundDuration = room.RoundDurationSeconds
            });

            // Notify guessers
            var maskedWord = _gameManager.GetMaskedWord(roomCode);
            await Clients.GroupExcept(roomCode, drawer.ConnectionId).SendAsync("RoundStarted", new
            {
                drawer = drawer.Username,
                wordLength = room.CurrentWord.Length,
                maskedWord = maskedWord,
                roundDuration = room.RoundDurationSeconds
            });
        }

        /// <summary>
        /// Clear the canvas
        /// </summary>
        public async Task ClearCanvas(string roomCode)
        {
            var room = _gameManager.GetRoom(roomCode);
            if (room == null) return;

            // Verify sender is drawer
            var player = room.Players.FirstOrDefault(p => p.ConnectionId == Context.ConnectionId);
            if (player == null || !player.IsDrawing) return;

            await Clients.Group(roomCode).SendAsync("ClearCanvas");
        }

        /// <summary>
        /// Handle disconnections
        /// </summary>
        public override async Task OnDisconnectedAsync(Exception? exception)
        {
            // Find which room the player was in
            foreach (var room in _gameManager.GetAllRooms())
            {
                var player = room.Players.FirstOrDefault(p => p.ConnectionId == Context.ConnectionId);
                
                if (player != null)
                {
                    var wasDrawing = player.IsDrawing;
                    
                    // Remove player
                    _gameManager.RemovePlayer(room.RoomCode, Context.ConnectionId);

                    // Notify others
                    var systemMessage = new ChatMessage
                    {
                        Username = "System",
                        Message = $"{player.Username} left the game",
                        IsSystemMessage = true
                    };

                    await Clients.Group(room.RoomCode).SendAsync("PlayerLeft", new
                    {
                        players = room.Players,
                        leftPlayer = player
                    });

                    await Clients.Group(room.RoomCode).SendAsync("ReceiveMessage", systemMessage);

                    // If drawer left, end the round
                    if (wasDrawing && room.Players.Count > 0)
                    {
                        await EndRound(room.RoomCode);
                    }

                    _logger.LogInformation($"{player.Username} disconnected from room {room.RoomCode}");
                    break;
                }
            }

            await base.OnDisconnectedAsync(exception);
        }
    }
}
```

**SignalR Concepts Explained:**

1. **Context.ConnectionId**: Unique ID for each connected client
2. **Groups**: Like chat rooms - broadcast to specific groups
3. **Clients.Caller**: Send to the client who called the method
4. **Clients.Group**: Send to everyone in a group
5. **Clients.OthersInGroup**: Send to group except caller
6. **Clients.Client**: Send to one specific client

---

## Step 7: Backend - Configuration

### Configure the Application

Edit `Program.cs`:

```csharp
using ScribbleGame.API.Hubs;
using ScribbleGame.API.Services;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add SignalR
builder.Services.AddSignalR();

// Register Game Manager as Singleton
// Singleton = one instance shared across all requests
builder.Services.AddSingleton<GameManager>();

// Configure CORS (Cross-Origin Resource Sharing)
// Required for frontend to connect from different port
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll",
        policy => policy
            .AllowAnyHeader()      // Allow any HTTP headers
            .AllowAnyMethod()      // Allow GET, POST, etc.
            .AllowCredentials()    // Required for SignalR
            .SetIsOriginAllowed(_ => true));  // Allow any origin (development only!)
});

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Enable CORS
app.UseCors("AllowAll");

app.UseAuthorization();

app.MapControllers();

// Map SignalR Hub
// This creates the /gamehub endpoint
app.MapHub<GameHub>("/gamehub");

app.Run();
```

**Configuration Explained:**

1. **AddSignalR()**: Registers SignalR services
2. **AddSingleton<GameManager>()**: One GameManager for entire app
3. **CORS**: Allows frontend (port 4200) to connect to backend (port 5001)
4. **MapHub<GameHub>("/gamehub")**: Creates SignalR endpoint at /gamehub

### Test the Backend

```bash
cd backend
dotnet run
```

You should see:
```
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:5001
      Now listening on: http://localhost:5000
```

**Backend is ready!** ✅

Keep it running and open a new terminal for frontend.

---

## Step 8: Frontend - Project Setup

### Create Angular Project

```bash
# Navigate to project root
cd scribble-game/frontend

# Create Angular project (select CSS and No SSR when prompted)
npx @angular/cli@17 new scribble-client --routing=true --style=css

# Navigate into project
cd scribble-client

# Install SignalR client library
npm install @microsoft/signalr

# Install RxJS (usually included, but let's be explicit)
npm install rxjs
```

**What Just Happened?**
- Created new Angular 17 project
- Enabled routing (for multiple pages)
- Installed SignalR client library
- Set up for TypeScript development

### Verify It Works

```bash
ng serve
```

Open browser to `http://localhost:4200` - you should see Angular welcome page!

Press `Ctrl+C` to stop for now.

---

## Step 9: Frontend - SignalR Service

### Why a Service?

Services in Angular:
- Encapsulate logic
- Reusable across components
- Singleton by default (one instance shared)
- Perfect for SignalR connection management

### Create SignalR Service

```bash
ng generate service services/signalr
```

Edit `src/app/services/signalr.service.ts`:

```typescript
import { Injectable } from '@angular/core';
import * as signalR from '@microsoft/signalr';
import { Subject } from 'rxjs';

// Define interfaces for type safety
export interface Player {
  connectionId: string;
  username: string;
  score: number;
  isDrawing: boolean;
  hasGuessedCorrectly: boolean;
}

export interface ChatMessage {
  username: string;
  message: string;
  timestamp: Date;
  isSystemMessage: boolean;
  isCorrectGuess: boolean;
}

export interface DrawingData {
  x: number;
  y: number;
  prevX: number;
  prevY: number;
  color: string;
  lineWidth: number;
  action: string;
}

@Injectable({
  providedIn: 'root'  // Singleton service
})
export class SignalrService {
  // SignalR connection instance
  private hubConnection!: signalR.HubConnection;

  // Observables for real-time events
  // Think of these as event emitters that components can subscribe to
  public roomCreated$ = new Subject<any>();
  public playerJoined$ = new Subject<any>();
  public playerLeft$ = new Subject<any>();
  public playersUpdated$ = new Subject<Player[]>();
  public receiveDrawing$ = new Subject<DrawingData>();
  public receiveMessage$ = new Subject<ChatMessage>();
  public roundStarted$ = new Subject<any>();
  public yourTurnToDraw$ = new Subject<any>();
  public correctGuess$ = new Subject<any>();
  public roundEnded$ = new Subject<any>();
  public clearCanvas$ = new Subject<void>();
  public error$ = new Subject<string>();

  constructor() {}

  /**
   * Start SignalR connection
   */
  async startConnection(): Promise<void> {
    // Build connection
    this.hubConnection = new signalR.HubConnectionBuilder()
      .withUrl('https://localhost:5001/gamehub', {
        skipNegotiation: false,
        transport: signalR.HttpTransportType.WebSockets | 
                   signalR.HttpTransportType.ServerSentEvents
      })
      .withAutomaticReconnect()  // Auto-reconnect on disconnect
      .build();

    // Set up event listeners
    this.setupEventListeners();

    try {
      // Actually connect
      await this.hubConnection.start();
      console.log('SignalR Connected');
    } catch (err) {
      console.error('Error connecting to SignalR:', err);
      throw err;
    }
  }

  /**
   * Set up all event listeners
   */
  private setupEventListeners(): void {
    // When server sends "RoomCreated" event
    this.hubConnection.on('RoomCreated', (data) => {
      this.roomCreated$.next(data);
    });

    // When server sends "PlayerJoined" event
    this.hubConnection.on('PlayerJoined', (data) => {
      this.playerJoined$.next(data);
    });

    // ... and so on for all events
    this.hubConnection.on('PlayerLeft', (data) => {
      this.playerLeft$.next(data);
    });

    this.hubConnection.on('PlayersUpdated', (players) => {
      this.playersUpdated$.next(players);
    });

    this.hubConnection.on('ReceiveDrawing', (data) => {
      this.receiveDrawing$.next(data);
    });

    this.hubConnection.on('ReceiveMessage', (message) => {
      this.receiveMessage$.next(message);
    });

    this.hubConnection.on('RoundStarted', (data) => {
      this.roundStarted$.next(data);
    });

    this.hubConnection.on('YourTurnToDraw', (data) => {
      this.yourTurnToDraw$.next(data);
    });

    this.hubConnection.on('CorrectGuess', (data) => {
      this.correctGuess$.next(data);
    });

    this.hubConnection.on('RoundEnded', (data) => {
      this.roundEnded$.next(data);
    });

    this.hubConnection.on('ClearCanvas', () => {
      this.clearCanvas$.next();
    });

    this.hubConnection.on('Error', (message) => {
      this.error$.next(message);
    });
  }

  /**
   * Call server methods
   */
  
  async createRoom(username: string): Promise<void> {
    await this.hubConnection.invoke('CreateRoom', username);
  }

  async joinRoom(roomCode: string, username: string): Promise<void> {
    await this.hubConnection.invoke('JoinRoom', roomCode, username);
  }

  async startGame(roomCode: string): Promise<void> {
    await this.hubConnection.invoke('StartGame', roomCode);
  }

  async sendDrawing(roomCode: string, data: DrawingData): Promise<void> {
    await this.hubConnection.invoke('SendDrawing', roomCode, data);
  }

  async sendMessage(roomCode: string, message: string): Promise<void> {
    await this.hubConnection.invoke('SendMessage', roomCode, message);
  }

  async endRound(roomCode: string): Promise<void> {
    await this.hubConnection.invoke('EndRound', roomCode);
  }

  async nextRound(roomCode: string): Promise<void> {
    await this.hubConnection.invoke('NextRound', roomCode);
  }

  async clearCanvas(roomCode: string): Promise<void> {
    await this.hubConnection.invoke('ClearCanvas', roomCode);
  }

  async disconnect(): Promise<void> {
    if (this.hubConnection) {
      await this.hubConnection.stop();
    }
  }
}
```

**Key Concepts:**

1. **Subject**: RxJS observable that can emit values
2. **invoke()**: Calls server methods
3. **on()**: Listens for server events
4. **withAutomaticReconnect()**: Handles connection drops

---

## Step 10: Frontend - Lobby Component

### Create Lobby Component

```bash
ng generate component lobby
```

This creates 4 files:
- lobby.component.ts (logic)
- lobby.component.html (template)
- lobby.component.css (styles)
- lobby.component.spec.ts (tests)

### Edit lobby.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { SignalrService } from '../services/signalr.service';

@Component({
  selector: 'app-lobby',
  templateUrl: './lobby.component.html',
  styleUrls: ['./lobby.component.css']
})
export class LobbyComponent implements OnInit {
  // Form fields
  username: string = '';
  roomCode: string = '';
  
  // Loading states
  isCreating: boolean = false;
  isJoining: boolean = false;
  errorMessage: string = '';
  isConnecting: boolean = false;

  constructor(
    private signalrService: SignalrService,
    private router: Router
  ) {}

  async ngOnInit(): Promise<void> {
    // Set up subscriptions to SignalR events
    this.setupSubscriptions();
    
    // Connect to SignalR when component loads
    this.isConnecting = true;
    try {
      await this.signalrService.startConnection();
      this.isConnecting = false;
    } catch (error) {
      this.isConnecting = false;
      this.errorMessage = 'Failed to connect to server. Please refresh the page.';
    }
  }

  /**
   * Subscribe to SignalR events
   */
  private setupSubscriptions(): void {
    // When room is created successfully
    this.signalrService.roomCreated$.subscribe((data) => {
      // Save to localStorage for the game component
      localStorage.setItem('roomCode', data.roomCode);
      localStorage.setItem('username', this.username);
      
      // Navigate to game page
      this.router.navigate(['/game']);
    });

    // When successfully joined a room
    this.signalrService.playerJoined$.subscribe((data) => {
      localStorage.setItem('roomCode', this.roomCode);
      localStorage.setItem('username', this.username);
      this.router.navigate(['/game']);
    });

    // Handle errors
    this.signalrService.error$.subscribe((error) => {
      this.errorMessage = error;
      this.isCreating = false;
      this.isJoining = false;
    });
  }

  /**
   * Create new room
   */
  async createRoom(): Promise<void> {
    // Validation
    if (!this.username.trim()) {
      this.errorMessage = 'Please enter a username';
      return;
    }

    this.errorMessage = '';
    this.isCreating = true;

    try {
      // Call SignalR method
      await this.signalrService.createRoom(this.username);
    } catch (error) {
      this.errorMessage = 'Failed to create room';
      this.isCreating = false;
    }
  }

  /**
   * Join existing room
   */
  async joinRoom(): Promise<void> {
    // Validation
    if (!this.username.trim()) {
      this.errorMessage = 'Please enter a username';
      return;
    }

    if (!this.roomCode.trim()) {
      this.errorMessage = 'Please enter a room code';
      return;
    }

    this.errorMessage = '';
    this.isJoining = true;

    try {
      await this.signalrService.joinRoom(this.roomCode.toUpperCase(), this.username);
    } catch (error) {
      this.errorMessage = 'Failed to join room';
      this.isJoining = false;
    }
  }
}
```

### Edit lobby.component.html

```html
<div class="lobby-container">
  <div class="lobby-card">
    <h1 class="game-title">🎨 Scribble Game</h1>
    <p class="subtitle">Draw, Guess, and Have Fun!</p>

    <!-- Loading spinner -->
    <div class="connection-status" *ngIf="isConnecting">
      <div class="spinner"></div>
      <p>Connecting to server...</p>
    </div>

    <!-- Main form -->
    <div class="form-container" *ngIf="!isConnecting">
      <!-- Username input -->
      <div class="input-group">
        <label for="username">Username</label>
        <input
          id="username"
          type="text"
          [(ngModel)]="username"
          placeholder="Enter your name"
          maxlength="20"
          (keyup.enter)="createRoom()"
        />
      </div>

      <!-- Room code input -->
      <div class="input-group">
        <label for="roomCode">Room Code (to join)</label>
        <input
          id="roomCode"
          type="text"
          [(ngModel)]="roomCode"
          placeholder="Enter room code"
          maxlength="6"
          (keyup.enter)="joinRoom()"
          style="text-transform: uppercase;"
        />
      </div>

      <!-- Error message -->
      <div class="error-message" *ngIf="errorMessage">
        {{ errorMessage }}
      </div>

      <!-- Action buttons -->
      <div class="button-group">
        <button
          class="btn btn-primary"
          (click)="createRoom()"
          [disabled]="isCreating || isJoining"
        >
          <span *ngIf="!isCreating">Create New Room</span>
          <span *ngIf="isCreating">Creating...</span>
        </button>

        <button
          class="btn btn-secondary"
          (click)="joinRoom()"
          [disabled]="isCreating || isJoining"
        >
          <span *ngIf="!isJoining">Join Room</span>
          <span *ngIf="isJoining">Joining...</span>
        </button>
      </div>

      <!-- Instructions -->
      <div class="instructions">
        <h3>How to Play:</h3>
        <ol>
          <li>Create a room or join with a room code</li>
          <li>Wait for other players to join</li>
          <li>Take turns drawing while others guess</li>
          <li>Earn points for correct guesses!</li>
        </ol>
      </div>
    </div>
  </div>
</div>
```

**Angular Template Syntax:**
- `[(ngModel)]`: Two-way data binding
- `*ngIf`: Conditional rendering
- `(click)`: Event binding
- `[disabled]`: Property binding

---
# 📚 Step-by-Step Tutorial: Building a Real-Time Multiplayer Drawing Game (Part 2)

## Continuing from Part 1...

---

## Step 11: Frontend - Game Component

### Create Game Component

```bash
ng generate component game
```

### Edit game.component.ts

```typescript
import { Component, OnInit, ViewChild, ElementRef, OnDestroy } from '@angular/core';
import { Router } from '@angular/router';
import { SignalrService, Player, ChatMessage, DrawingData } from '../services/signalr.service';

@Component({
  selector: 'app-game',
  templateUrl: './game.component.html',
  styleUrls: ['./game.component.css']
})
export class GameComponent implements OnInit, OnDestroy {
  // Get reference to canvas element
  @ViewChild('canvas', { static: true }) canvasRef!: ElementRef<HTMLCanvasElement>;
  
  // Canvas context for drawing
  private ctx!: CanvasRenderingContext2D;
  
  // Drawing state
  private isDrawing = false;
  private lastX = 0;
  private lastY = 0;

  // Game state
  roomCode: string = '';
  username: string = '';
  players: Player[] = [];
  chatMessages: ChatMessage[] = [];
  currentMessage: string = '';
  
  // Round info
  isMyTurn = false;
  currentWord: string = '';
  maskedWord: string = '';
  currentDrawer: string = '';
  
  // Game flow
  gameStarted = false;
  roundEnded = false;
  roundEndData: any = null;
  
  // Drawing tools
  selectedColor: string = '#000000';
  selectedLineWidth: number = 2;
  
  colors = ['#000000', '#FFFFFF', '#FF0000', '#00FF00', '#0000FF', 
            '#FFFF00', '#FF00FF', '#00FFFF', '#FFA500', '#800080'];
  lineWidths = [2, 5, 10, 15];

  // Timer
  timeRemaining: number = 0;
  private timerInterval: any;

  constructor(
    private signalrService: SignalrService,
    private router: Router
  ) {}

  async ngOnInit(): Promise<void> {
    // Get room info from localStorage (set in lobby)
    this.roomCode = localStorage.getItem('roomCode') || '';
    this.username = localStorage.getItem('username') || '';

    // If no room info, redirect to lobby
    if (!this.roomCode || !this.username) {
      this.router.navigate(['/']);
      return;
    }

    // Initialize canvas and subscriptions
    this.setupCanvas();
    this.setupSubscriptions();
  }

  ngOnDestroy(): void {
    // Clean up timer when component is destroyed
    if (this.timerInterval) {
      clearInterval(this.timerInterval);
    }
  }

  /**
   * Initialize canvas
   */
  private setupCanvas(): void {
    const canvas = this.canvasRef.nativeElement;
    this.ctx = canvas.getContext('2d')!;
    
    // Set line style
    this.ctx.lineCap = 'round';   // Rounded line ends
    this.ctx.lineJoin = 'round';  // Rounded corners
  }

  /**
   * Subscribe to all SignalR events
   */
  private setupSubscriptions(): void {
    // Player joined
    this.signalrService.playerJoined$.subscribe((data) => {
      this.players = data.players;
      this.addSystemMessage(`${data.newPlayer.username} joined the game`);
    });

    // Player left
    this.signalrService.playerLeft$.subscribe((data) => {
      this.players = data.players;
      this.addSystemMessage(`${data.leftPlayer.username} left the game`);
    });

    // Scores updated
    this.signalrService.playersUpdated$.subscribe((players) => {
      this.players = players;
    });

    // Round started (for guessers)
    this.signalrService.roundStarted$.subscribe((data) => {
      this.gameStarted = true;
      this.roundEnded = false;
      this.isMyTurn = false;
      this.currentDrawer = data.drawer;
      this.maskedWord = data.maskedWord;
      this.timeRemaining = data.roundDuration;
      this.startTimer();
      this.addSystemMessage(`${data.drawer} is drawing!`);
    });

    // Your turn to draw
    this.signalrService.yourTurnToDraw$.subscribe((data) => {
      this.gameStarted = true;
      this.roundEnded = false;
      this.isMyTurn = true;
      this.currentWord = data.word;
      this.timeRemaining = data.roundDuration;
      this.startTimer();
      this.addSystemMessage(`Your turn! Draw: ${data.word}`);
    });

    // Receive drawing data
    this.signalrService.receiveDrawing$.subscribe((data) => {
      this.drawOnCanvas(data);
    });

    // Receive chat message
    this.signalrService.receiveMessage$.subscribe((message) => {
      this.chatMessages.push(message);
      this.scrollChatToBottom();
    });

    // Correct guess
    this.signalrService.correctGuess$.subscribe((data) => {
      // Player list will be updated via playersUpdated$
    });

    // Round ended
    this.signalrService.roundEnded$.subscribe((data) => {
      this.roundEnded = true;
      this.roundEndData = data;
      this.isMyTurn = false;
      this.stopTimer();
      this.addSystemMessage(`Round ended! The word was: ${data.word}`);
    });

    // Clear canvas
    this.signalrService.clearCanvas$.subscribe(() => {
      this.clearCanvas();
    });

    // Errors
    this.signalrService.error$.subscribe((error) => {
      alert(error);
    });
  }

  /**
   * Start countdown timer
   */
  private startTimer(): void {
    if (this.timerInterval) {
      clearInterval(this.timerInterval);
    }

    this.timerInterval = setInterval(() => {
      this.timeRemaining--;
      if (this.timeRemaining <= 0) {
        this.stopTimer();
        if (this.isMyTurn) {
          this.endRound();
        }
      }
    }, 1000);  // Update every second
  }

  private stopTimer(): void {
    if (this.timerInterval) {
      clearInterval(this.timerInterval);
    }
  }

  /**
   * Start the game
   */
  async startGame(): Promise<void> {
    await this.signalrService.startGame(this.roomCode);
  }

  /**
   * End current round
   */
  async endRound(): Promise<void> {
    await this.signalrService.endRound(this.roomCode);
  }

  /**
   * Start next round
   */
  async nextRound(): Promise<void> {
    await this.signalrService.nextRound(this.roomCode);
  }

  /**
   * Mouse down - start drawing
   */
  onMouseDown(event: MouseEvent): void {
    if (!this.isMyTurn) return;
    
    this.isDrawing = true;
    const rect = this.canvasRef.nativeElement.getBoundingClientRect();
    this.lastX = event.clientX - rect.left;
    this.lastY = event.clientY - rect.top;
  }

  /**
   * Mouse move - draw line
   */
  onMouseMove(event: MouseEvent): void {
    if (!this.isDrawing || !this.isMyTurn) return;

    const rect = this.canvasRef.nativeElement.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;

    // Create drawing data
    const drawingData: DrawingData = {
      x: x,
      y: y,
      prevX: this.lastX,
      prevY: this.lastY,
      color: this.selectedColor,
      lineWidth: this.selectedLineWidth,
      action: 'draw'
    };

    // Draw on own canvas
    this.drawOnCanvas(drawingData);
    
    // Send to other players
    this.signalrService.sendDrawing(this.roomCode, drawingData);

    // Update last position
    this.lastX = x;
    this.lastY = y;
  }

  /**
   * Mouse up - stop drawing
   */
  onMouseUp(): void {
    this.isDrawing = false;
  }

  /**
   * Mouse leave - stop drawing
   */
  onMouseLeave(): void {
    this.isDrawing = false;
  }

  /**
   * Draw line on canvas
   */
  private drawOnCanvas(data: DrawingData): void {
    this.ctx.strokeStyle = data.color;
    this.ctx.lineWidth = data.lineWidth;
    
    this.ctx.beginPath();
    this.ctx.moveTo(data.prevX, data.prevY);
    this.ctx.lineTo(data.x, data.y);
    this.ctx.stroke();
  }

  /**
   * Clear canvas
   */
  clearCanvas(): void {
    const canvas = this.canvasRef.nativeElement;
    this.ctx.clearRect(0, 0, canvas.width, canvas.height);
  }

  /**
   * Clear canvas button click
   */
  async clearCanvasClick(): Promise<void> {
    if (!this.isMyTurn) return;
    this.clearCanvas();
    await this.signalrService.clearCanvas(this.roomCode);
  }

  /**
   * Select color
   */
  selectColor(color: string): void {
    this.selectedColor = color;
  }

  /**
   * Select brush size
   */
  selectLineWidth(width: number): void {
    this.selectedLineWidth = width;
  }

  /**
   * Send chat message
   */
  async sendMessage(): Promise<void> {
    if (!this.currentMessage.trim()) return;

    await this.signalrService.sendMessage(this.roomCode, this.currentMessage);
    this.currentMessage = '';
  }

  /**
   * Add system message to chat
   */
  private addSystemMessage(message: string): void {
    this.chatMessages.push({
      username: 'System',
      message: message,
      timestamp: new Date(),
      isSystemMessage: true,
      isCorrectGuess: false
    });
    this.scrollChatToBottom();
  }

  /**
   * Scroll chat to bottom
   */
  private scrollChatToBottom(): void {
    setTimeout(() => {
      const chatBox = document.querySelector('.chat-messages');
      if (chatBox) {
        chatBox.scrollTop = chatBox.scrollHeight;
      }
    }, 100);
  }

  /**
   * Copy room code to clipboard
   */
  copyRoomCode(): void {
    navigator.clipboard.writeText(this.roomCode);
    alert('Room code copied to clipboard!');
  }

  /**
   * Leave room
   */
  leaveRoom(): void {
    if (confirm('Are you sure you want to leave?')) {
      this.signalrService.disconnect();
      localStorage.removeItem('roomCode');
      localStorage.removeItem('username');
      this.router.navigate(['/']);
    }
  }

  /**
   * Get players sorted by score
   */
  getPlayersSorted(): Player[] {
    return [...this.players].sort((a, b) => b.score - a.score);
  }
}
```

**Component Explained:**

1. **ViewChild**: Gets reference to canvas element
2. **Canvas Context**: Used for drawing operations
3. **Mouse Events**: Track drawing movements
4. **Timer**: Countdown for each round
5. **SignalR Subscriptions**: Listen to all game events

---

## Step 12: Frontend - Canvas Drawing

### Understanding Canvas Drawing

**How Canvas Works:**
```
Canvas is like a digital whiteboard
- Pixel-based (800x500 pixels in our case)
- Draw with "context" (like a brush)
- No built-in "undo" - must redraw everything
```

**Drawing a Line:**
```typescript
ctx.beginPath();           // Start new path
ctx.moveTo(x1, y1);       // Move to start point
ctx.lineTo(x2, y2);       // Draw line to end point
ctx.stroke();              // Actually draw it
```

### Mouse Coordinate Calculation

**Problem:** Mouse events give us coordinates relative to the page, but we need coordinates relative to the canvas.

**Solution:**
```typescript
const rect = canvas.getBoundingClientRect();
const canvasX = mouseEvent.clientX - rect.left;
const canvasY = mouseEvent.clientY - rect.top;
```

### Drawing Flow

```
1. Mouse Down → Set isDrawing = true, store position
2. Mouse Move → If drawing, draw line from last to current position
3. Mouse Up → Set isDrawing = false
4. Send coordinates to server via SignalR
5. Server broadcasts to other players
6. Other players draw the same line
```

---

## Step 13: Frontend - Styling

### Global Styles

Edit `src/styles.css`:

```css
/* Reset and base styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Custom scrollbar */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
```

### Lobby Styles

Edit `src/app/lobby/lobby.component.css`:

```css
.lobby-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.lobby-card {
  background: white;
  border-radius: 20px;
  padding: 40px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-width: 500px;
  width: 100%;
  animation: slideIn 0.5s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.game-title {
  font-size: 2.5rem;
  margin: 0 0 10px 0;
  color: #667eea;
  text-align: center;
}

.subtitle {
  text-align: center;
  color: #666;
  margin: 0 0 30px 0;
  font-size: 1.1rem;
}

/* Connection status */
.connection-status {
  text-align: center;
  padding: 40px 0;
}

.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #667eea;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Form elements */
.form-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.input-group label {
  font-weight: 600;
  color: #333;
  font-size: 0.9rem;
}

.input-group input {
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 1rem;
  transition: all 0.3s;
}

.input-group input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

/* Buttons */
.button-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 10px;
}

.btn {
  padding: 14px 24px;
  border: none;
  border-radius: 10px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(102, 126, 234, 0.4);
}

.btn-secondary {
  background: white;
  color: #667eea;
  border: 2px solid #667eea;
}

.btn-secondary:hover:not(:disabled) {
  background: #667eea;
  color: white;
  transform: translateY(-2px);
}

/* Error message */
.error-message {
  background: #fee;
  color: #c33;
  padding: 12px;
  border-radius: 8px;
  text-align: center;
  font-size: 0.9rem;
}

/* Instructions */
.instructions {
  margin-top: 30px;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 10px;
}

.instructions h3 {
  margin: 0 0 15px 0;
  color: #333;
  font-size: 1.1rem;
}

.instructions ol {
  margin: 0;
  padding-left: 20px;
  color: #666;
}

.instructions li {
  margin-bottom: 8px;
}

/* Responsive */
@media (max-width: 600px) {
  .lobby-card {
    padding: 30px 20px;
  }

  .game-title {
    font-size: 2rem;
  }
}
```

**CSS Concepts:**

1. **Flexbox**: Modern layout system
2. **Gradients**: Smooth color transitions
3. **Transitions**: Smooth animations
4. **Box-shadow**: Depth and elevation
5. **Media queries**: Responsive design

---

## Step 14: Testing the Application

### Test Plan

**1. Start Backend**
```bash
cd backend
dotnet run
```

**2. Start Frontend (new terminal)**
```bash
cd frontend
ng serve
```

**3. Open Multiple Browsers**
- Chrome: `http://localhost:4200`
- Firefox: `http://localhost:4200`
- Or use incognito windows

**4. Test Scenarios**

#### Test 1: Create and Join Room
```
Browser 1:
1. Enter username "Alice"
2. Click "Create New Room"
3. Note the room code (e.g., "ABC123")

Browser 2:
1. Enter username "Bob"
2. Enter room code "ABC123"
3. Click "Join Room"
4. Verify both players see each other
```

#### Test 2: Start Game
```
Browser 1:
1. Click "Start Game"
2. Verify drawer sees actual word
3. Verify guesser sees masked word
```

#### Test 3: Drawing
```
Browser 1 (Drawer):
1. Select color (e.g., red)
2. Select brush size (e.g., 10px)
3. Draw on canvas
4. Verify Browser 2 sees the drawing in real-time
```

#### Test 4: Guessing
```
Browser 2 (Guesser):
1. Type the correct word in chat
2. Verify "Correct guess" message appears
3. Verify score increases
4. Verify player list updates
```

#### Test 5: Round Flow
```
1. Wait for all players to guess or time to expire
2. Verify round ends automatically
3. Click "Next Round"
4. Verify next player becomes drawer
5. Verify canvas clears
```

#### Test 6: Disconnect Handling
```
Browser 2:
1. Close the browser tab
2. Verify Browser 1 sees "Bob left the game"
3. Verify player count updates
```

### Common Issues and Solutions

**Issue**: Cannot connect to SignalR
```
Solution:
- Verify backend is running on port 5001
- Check browser console for errors
- Ensure CORS is configured
```

**Issue**: Drawing not syncing
```
Solution:
- Verify you're the drawer (check isMyTurn)
- Check network tab for WebSocket frames
- Ensure SignalR connection is established
```

**Issue**: Chat messages not appearing
```
Solution:
- Check signalrService.receiveMessage$ subscription
- Verify messages are being sent to server
- Check for JavaScript errors in console
```

---

## Step 15: Deployment

### Option 1: Docker Deployment (Recommended for Testing)

**1. Create docker-compose.yml** (already provided in project)

**2. Build and run:**
```bash
docker-compose up --build
```

**3. Access:**
- Backend: `http://localhost:5001`
- Frontend: `http://localhost:4200`

### Option 2: Azure Deployment (Production)

**Step-by-step Azure Deployment:**

#### Deploy Backend to Azure App Service

**1. Install Azure CLI**
```bash
# Windows: Download from microsoft.com
# Mac: brew install azure-cli
# Linux: curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

**2. Login to Azure**
```bash
az login
```

**3. Create Resource Group**
```bash
az group create --name scribble-rg --location eastus
```

**4. Create App Service Plan**
```bash
az appservice plan create \
  --name scribble-plan \
  --resource-group scribble-rg \
  --sku B1 \
  --is-linux
```

**5. Create Web App**
```bash
az webapp create \
  --name scribble-backend-YOUR-NAME \
  --resource-group scribble-rg \
  --plan scribble-plan \
  --runtime "DOTNET|8.0"
```

**6. Deploy Code**
```bash
cd backend
az webapp up \
  --name scribble-backend-YOUR-NAME \
  --resource-group scribble-rg
```

**7. Configure CORS in Azure Portal**
- Go to Azure Portal
- Find your App Service
- Navigate to "CORS"
- Add your frontend URL
- Enable "Access-Control-Allow-Credentials"

#### Deploy Frontend to Azure Static Web App

**1. Build Frontend**
```bash
cd frontend
npm run build
```

**2. Create Static Web App**
```bash
az staticwebapp create \
  --name scribble-frontend \
  --resource-group scribble-rg \
  --source https://github.com/your-repo \
  --location eastus \
  --branch main \
  --app-location "/frontend" \
  --output-location "dist/scribble-client"
```

**3. Update SignalR URL**

Edit `src/app/services/signalr.service.ts`:
```typescript
.withUrl('https://scribble-backend-YOUR-NAME.azurewebsites.net/gamehub')
```

**4. Rebuild and deploy**
```bash
npm run build
# Deploy via Azure Static Web Apps GitHub Action
```

### Option 3: Self-Hosted (VPS)

**Requirements:**
- Linux server (Ubuntu recommended)
- Nginx
- .NET 8.0 Runtime
- Node.js

**Steps:**

**1. Install .NET Runtime**
```bash
wget https://dot.net/v1/dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 8.0 --runtime aspnetcore
```

**2. Install Nginx**
```bash
sudo apt update
sudo apt install nginx
```

**3. Build and Deploy Backend**
```bash
cd backend
dotnet publish -c Release -o /var/www/scribble-backend
```

**4. Create systemd service**
```bash
sudo nano /etc/systemd/system/scribble.service
```

```ini
[Unit]
Description=Scribble Game Backend

[Service]
WorkingDirectory=/var/www/scribble-backend
ExecStart=/usr/local/bin/dotnet /var/www/scribble-backend/ScribbleGame.API.dll
Restart=always
RestartSec=10
SyslogIdentifier=scribble
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production

[Install]
WantedBy=multi-user.target
```

**5. Configure Nginx**
```bash
sudo nano /etc/nginx/sites-available/scribble
```

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location /gamehub {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        root /var/www/scribble-frontend;
        try_files $uri $uri/ /index.html;
    }
}
```

**6. Enable and start**
```bash
sudo systemctl enable scribble
sudo systemctl start scribble
sudo systemctl enable nginx
sudo systemctl restart nginx
```

---

## Congratulations! 🎉

You've built a complete real-time multiplayer drawing game!

### What You've Learned

**Backend:**
- ✅ ASP.NET Core Web API
- ✅ SignalR for WebSockets
- ✅ Dependency Injection
- ✅ Service architecture
- ✅ Thread-safe collections

**Frontend:**
- ✅ Angular framework
- ✅ TypeScript
- ✅ Component architecture
- ✅ RxJS observables
- ✅ Canvas API
- ✅ Real-time communication

**DevOps:**
- ✅ Docker containerization
- ✅ Azure deployment
- ✅ CORS configuration
- ✅ Production best practices

### Next Steps

**Immediate Improvements:**
1. Add more words to the pool
2. Implement word categories
3. Add difficulty levels
4. Create custom themes

**Advanced Features:**
1. Database persistence (Entity Framework)
2. User accounts (JWT authentication)
3. Private rooms with passwords
4. Chat history
5. Drawing tools (eraser, fill, shapes)
6. Mobile app

**Production Enhancements:**
1. Rate limiting
2. Input sanitization
3. Error logging (Application Insights)
4. Performance monitoring
5. Auto-scaling
6. CDN for static assets

---

## Resources

**Documentation:**
- [ASP.NET Core Docs](https://docs.microsoft.com/aspnet/core)
- [SignalR Docs](https://docs.microsoft.com/aspnet/core/signalr)
- [Angular Docs](https://angular.io/docs)
- [Canvas API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)

**Community:**
- Stack Overflow
- GitHub Issues
- Reddit: r/dotnet, r/Angular
- Discord servers

---

## Troubleshooting Guide

### Backend Issues

**Port already in use:**
```bash
# Find process
netstat -ano | findstr :5001
# Kill it (Windows)
taskkill /PID <PID> /F
# Kill it (Linux/Mac)
kill -9 <PID>
```

**SignalR not connecting:**
- Check CORS configuration
- Verify hub URL is correct
- Ensure WebSocket support enabled

### Frontend Issues

**npm install fails:**
```bash
rm -rf node_modules
npm cache clean --force
npm install
```

**TypeScript errors:**
```bash
npm install --save-dev @types/node
```

**Build fails:**
```bash
ng build --configuration production
# Check for errors
```

### Deployment Issues

**Azure deployment fails:**
- Check resource quotas
- Verify App Service plan supports Linux
- Ensure correct runtime version

**Docker container crashes:**
```bash
docker logs <container-id>
# Check logs for errors
```

---

## Final Thoughts

Building real-time applications is challenging but rewarding. You now have a solid foundation in:

- WebSocket programming
- Full-stack development
- Modern web frameworks
- Cloud deployment

Keep building, keep learning, and most importantly, have fun! 🚀

---

**Happy Coding!** 🎨👨‍💻👩‍💻
