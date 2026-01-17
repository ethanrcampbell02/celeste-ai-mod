# Celeste AI Mod

An Everest mod that enables AI agent control and state extraction for Celeste gameplay, designed to work with reinforcement learning training environments.

## Overview

This mod extends Celeste with AI capabilities, providing a TCP-based interface for external AI agents to control the player character and extract game state information. It's the companion mod to the `celeste-ai-gym` Python training environment.

## Features

- **TCP Server Interface**: Communicates with external AI training environments
- **Real-time Control**: AI agents can control player input programmatically
- **State Extraction**: Provides detailed game state information including position, velocity, room data
- **Screenshot Capture**: Real-time game screen capture for visual observations
- **Automatic Cutscene Skipping**: Streamlines training by bypassing non-gameplay content
- **Reset Functionality**: Supports environment resets for episodic training

## Architecture

```
External AI Agent (Python)
    ↕ TCP Socket (port 5000)
Celeste AI Mod (C#)
    ↕ Everest Mod Framework
Celeste Game Engine
```

## Installation

### Prerequisites

- Celeste (Steam/itch.io version)
- [Everest](https://everestapi.github.io/) mod loader
- [ProgrammaticInput](https://github.com/max4805/ProgrammaticInput) dependency mod

### Installation Steps

1. **Install Everest** (if not already installed)
   - Download from [Everest website](https://everestapi.github.io/)
   - Run the installer and follow instructions

2. **Install ProgrammaticInput dependency**
   - Download from GameBanana or compile from source
   - Place in your Celeste `Mods` folder

3. **Install this mod**
   - Copy the mod files to your Celeste `Mods` folder
   - The structure should be:
     ```
     Celeste/
     └── Mods/
         └── celeste-ai-mod/
             ├── everest.yaml
             ├── bin/
             │   └── CelesteAI.dll
             └── Source/ (if compiling from source)
     ```

## Building from Source

### Prerequisites

- .NET 8.0 SDK
- Visual Studio 2022 or VS Code with C# extension

### Build Steps

```bash
# Clone the repository
git clone <repository-url>
cd celeste-ai-mod

# Restore dependencies
dotnet restore

# Build the project
dotnet build --configuration Release

# The built mod will be in bin/Release/
```

## Configuration

### Mod Settings

The mod can be configured through the Celeste mod options menu:

- **Enable AI Control**: Toggle AI takeover of player input
- **TCP Port**: Configure the communication port (default: 5000)
- **Auto-skip Cutscenes**: Automatically skip story elements
- **Debug Mode**: Enable verbose logging for troubleshooting

### Everest Configuration

The mod dependencies are defined in `everest.yaml`:

```yaml
- Name: CelesteAI
  Version: 1.0.0
  DLL: bin/CelesteAI.dll
  Dependencies:
    - Name: EverestCore
      Version: 1.5577.0
    - Name: ProgrammaticInput
      Version: 0.1.3
```

## Usage

### Starting the Server

1. Launch Celeste with Everest
2. Enable the AI mod in settings
3. The mod automatically starts a TCP server on port 5000
4. Load any level to begin AI interaction

### TCP Communication Protocol

The mod communicates using JSON messages:

#### Incoming Messages (from AI agent)

**Reset Request**
```json
{
  "type": "reset"
}
```

**Action Input**
```json
{
  "type": "action", 
  "actions": [0, 0, 1, 0, 1, 0, 0]
}
```
*Actions array: [up, down, left, right, jump, dash, grab]*

#### Outgoing Messages (to AI agent)

**Game State**
```json
{
  "type": "state",
  "image": "base64_encoded_screenshot",
  "gameState": {
    "position": [1245.6, 567.8],
    "velocity": [120.0, -45.3],
    "room": "1-ForsakenCity/a-01",
    "distance": 1250.75,
    "isDead": false,
    "completedLevel": false,
    "canDash": true,
    "canGrab": true,
    "facing": "right"
  }
}
```

## Integration with AI Training

### With celeste-ai-gym

The mod is designed to work seamlessly with the `celeste-ai-gym` Python environment:

```python
from CelesteEnv import CelesteEnv

# Create environment (connects to this mod)
env = CelesteEnv()

# Use standard Gym interface
obs = env.reset()
action = env.action_space.sample()
obs, reward, done, info = env.step(action)
```

### Custom Integration

For custom AI frameworks, implement TCP communication:

```python
import socket
import json
import base64

# Connect to mod
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('127.0.0.1', 5000))

# Send action
action_msg = {"type": "action", "actions": [0, 0, 1, 0, 0, 0, 0]}
sock.send(json.dumps(action_msg).encode())

# Receive state
response = sock.recv(524288)  # Large buffer for image data
state_data = json.loads(response.decode())
```

## Game State Details

### Position & Physics
- **position**: Player coordinates in level space
- **velocity**: Current movement velocity vector
- **facing**: Player facing direction ("left" or "right")

### Player Abilities
- **canDash**: Whether dash ability is available
- **canGrab**: Whether grab/climb is possible
- **onGround**: Whether player is on solid ground

### Level Information
- **room**: Current room identifier
- **distance**: Progress metric through the level
- **completedLevel**: Whether level is finished

### Status Flags
- **isDead**: Whether player has died
- **inCutscene**: Whether in a story sequence

## File Structure

```
celeste-ai-mod/
├── everest.yaml           # Mod metadata and dependencies
├── CelesteAI.sln         # Visual Studio solution
├── Source/               # C# source code
│   ├── CelesteAI.csproj  # Project file
│   ├── MadelAIneModule.cs       # Main mod class
│   ├── MadelAIneModuleSettings.cs # Mod configuration
│   ├── MadelAIneModuleSession.cs  # Session state
│   ├── MadelAIneModuleSaveData.cs # Persistent data
│   └── GameState.cs      # Game state extraction
├── bin/                  # Compiled mod output
└── README.md            # This file
```

## Development

### Key Classes

**MadelAIneModule**: Main mod entry point and TCP server management
**GameState**: Extracts and serializes game state information  
**ModuleSettings**: Configuration and user preferences
**ModuleSession**: Per-level session management

### Adding Features

To extend the mod functionality:

1. Add new game state properties to `GameState.cs`
2. Update JSON serialization in state extraction
3. Modify TCP message handling in `MadelAIneModule.cs`
4. Update protocol documentation

### Debugging

Enable debug mode in mod settings for verbose logging:
- TCP connection events
- Message parsing details  
- Game state extraction timing
- Error diagnostics

## Performance Considerations

- **Screenshot Frequency**: Balance visual fidelity with performance
- **TCP Buffer Sizes**: Optimize for your network setup
- **State Extraction**: Minimize overhead on game loop
- **Memory Usage**: Monitor for memory leaks during long training

## Compatibility

### Celeste Versions
- Tested with Celeste v1.4.0.0+
- Requires Everest 1.5577.0+

### Other Mods
- Generally compatible with gameplay mods
- May conflict with mods that override input handling
- Compatible with debug/speedrun tools

### Platforms
- Windows (primary support)
- Linux (via Mono, limited testing)
- macOS (via Mono, experimental)

## Troubleshooting

### Common Issues

**TCP Connection Failed**
- Check firewall settings
- Verify port 5000 availability
- Ensure mod is enabled in settings

**Game Freezes**
- Reduce screenshot resolution
- Check for infinite loops in AI agent
- Monitor memory usage

**Input Not Working**
- Verify ProgrammaticInput dependency
- Check action message format
- Ensure proper JSON encoding

## Contributing

1. Fork the repository
2. Create a feature branch
3. Follow C# coding conventions
4. Test with both simple and complex levels
5. Submit a pull request

## License

[Specify your license here]

## Credits

- Everest team for the modding framework
- max4805 for ProgrammaticInput foundation
- viddie for Physics Inspector reference code
- Celeste community for support and feedback

## Related Projects

- [Celeste AI Gym](../celeste-ai-gym): Python RL training environment
- [ProgrammaticInput](https://github.com/max4805/ProgrammaticInput): Input automation framework
- [Everest](https://everestapi.github.io/): Celeste mod loader