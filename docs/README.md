# Paradigm Framework

## Overview

Paradigm is a comprehensive framework for Arma 3 mission development that provides core functionality for building complex, persistent multiplayer scenarios. It serves as the foundation for the Mike Force mission system and can be used to create other advanced mission types.

## Architecture

### Client-Server Model
Paradigm follows a distributed architecture with clear separation between client and server functionality:

- **Server** (`server/`) - Mission state management, AI control, database operations
- **Client** (`client/`) - User interface, input handling, visual effects
- **Global** (`global/`) - Shared functionality accessible from both client and server

### Component Structure
```
Paradigm/
├── client/          # Client-side functionality
│   ├── functions/   # Client functions (UI, input, visuals)
│   └── configs/     # Client configuration files
├── server/          # Server-side functionality  
│   ├── functions/   # Server functions (AI, persistence, logic)
│   └── configs/     # Server configuration files
├── global/          # Shared functionality
│   ├── functions/   # Global functions (events, utilities)
│   └── config/      # Global configuration
└── docs/            # Documentation
    └── systems/     # System-specific documentation
```

## Core Systems

### Event System
A lightweight publish-subscribe event system enabling decoupled communication between mission components.

**Key Features:**
- Event dispatching and handler registration
- Support for custom events
- Built-in events for common mission activities

**Documentation:** [Event System](systems/events.md)

### Database System
Persistent data storage and retrieval system for maintaining mission state across server restarts.

**Key Features:**
- Profile-based data storage
- Automatic data serialization
- Cross-session persistence

### AI Management
Comprehensive AI spawning, behavior, and lifecycle management.

**Key Features:**
- Dynamic AI group creation
- Behavioral AI scripting
- Performance-optimized AI cleanup

### User Interface Framework
Standardized UI components and interaction systems.

**Key Features:**
- Reusable UI dialogs and displays
- Input handling and validation
- Responsive design principles

### Networking Utilities
Remote execution helpers and network synchronization tools.

**Key Features:**
- Safe remote execution wrappers
- Data synchronization helpers
- Network event propagation

## Function Naming Convention

Paradigm uses a consistent naming convention for all functions:

```
para_<scope>_fnc_<function_name>
```

Where:
- `para` - Framework prefix
- `<scope>` - Function scope:
  - `s` - Server-side only
  - `c` - Client-side only  
  - `g` - Global (shared)
- `fnc` - Function identifier
- `<function_name>` - Descriptive function name

**Examples:**
- `para_s_fnc_spawn_ai` - Server function for AI spawning
- `para_c_fnc_show_dialog` - Client function for dialog display
- `para_g_fnc_event_dispatch` - Global function for event dispatching

## Configuration Management

### File Structure
Configuration files are organized by scope and purpose:

- **System Config** - Core framework settings
- **Component Config** - Individual component configurations
- **Mission Config** - Mission-specific overrides

### Include System
Paradigm uses a hierarchical include system for configuration management:

```cpp
// Main config includes component configs
#include "client\config.hpp"
#include "server\config.hpp" 
#include "global\config.hpp"

// Component configs include specific functionality
#include "functions.hpp"
#include "configs\notifications.hpp"
#include "configs\ui\dialogs.hpp"
```

## Development Workflow

### Function Development
1. **Identify Scope** - Determine if function is client, server, or global
2. **Create Function File** - Place in appropriate scope directory
3. **Register Function** - Add to `functions.hpp` for the scope
4. **Document Function** - Include header comments and parameter documentation

### Testing
1. **Unit Testing** - Test individual functions in isolation
2. **Integration Testing** - Test component interactions
3. **Mission Testing** - Test in full mission environment

### Integration
1. **Add to Build** - Include new files in build configuration
2. **Update Documentation** - Document new functionality
3. **Version Control** - Commit changes with descriptive messages

## API Reference

### Core Functions

#### Global Functions (`para_g_fnc_*`)
- `para_g_fnc_event_add_handler` - Register event handlers
- `para_g_fnc_event_dispatch` - Dispatch events to handlers
- `para_g_fnc_utils_*` - Various utility functions

#### Server Functions (`para_s_fnc_*`)
- `para_s_fnc_profile_db` - Database operations
- `para_s_fnc_ai_*` - AI management functions
- `para_s_fnc_spawn_*` - Object spawning functions

#### Client Functions (`para_c_fnc_*`)
- `para_c_fnc_ui_*` - User interface functions
- `para_c_fnc_input_*` - Input handling functions
- `para_c_fnc_display_*` - Display management functions

## Integration Guide

### Using Paradigm in Your Mission

1. **Include Framework**
   ```cpp
   // In mission's config.cpp
   #include "Paradigm\config.cpp"
   ```

2. **Initialize Framework**
   ```sqf
   // In mission init
   [] call para_g_fnc_init;
   ```

3. **Use Framework Functions**
   ```sqf
   // Register for events
   ["vehicleCreated", {
       params ["_args", "_vehicle"];
       // Handle vehicle creation
   }] call para_g_fnc_event_add_handler;
   
   // Use database functions
   ["playerData", _playerData] call para_s_fnc_profile_db;
   ```

### Extending Paradigm

1. **Create Custom Functions**
   - Follow naming convention
   - Place in appropriate scope directory
   - Register in functions.hpp

2. **Add Custom Events**
   - Use descriptive event names
   - Document event parameters
   - Dispatch events at appropriate times

3. **Integrate with Existing Systems**
   - Use existing events when possible
   - Follow established patterns
   - Maintain backward compatibility

## Performance Considerations

### Function Execution
- Server functions execute on server machine only
- Client functions execute on each player's machine
- Global functions execute where called

### Event System
- Events are local to executing machine
- Use `remoteExec` for cross-network event propagation
- Keep event handlers lightweight

### Database Operations
- Database operations are server-side only
- Minimize frequent database writes
- Use batch operations when possible

### Memory Management
- Clean up created objects and variables
- Use local variables when possible
- Monitor memory usage in complex scenarios

## Troubleshooting

### Common Issues

#### Function Not Found
- Check function registration in `functions.hpp`
- Verify correct scope prefix (`para_s_`, `para_c_`, `para_g_`)
- Ensure framework is properly initialized

#### Event Not Firing
- Verify event name spelling
- Check handler registration timing
- Confirm event is being dispatched

#### Performance Issues
- Profile heavy functions
- Check for infinite loops in event handlers
- Monitor AI count and cleanup

### Debugging Tools

#### Logging
```sqf
// Enable debug logging
para_debug_enabled = true;

// Custom logging
diag_log format ["[Paradigm] %1", _message];
```

#### Event Monitoring
```sqf
// Monitor all events
["*", {
    params ["_args", "_eventName"];
    diag_log format ["Event: %1, Args: %2", _eventName, _args];
}] call para_g_fnc_event_add_handler;
```

## Contributing

### Code Standards
- Follow existing naming conventions
- Include comprehensive documentation
- Write defensive code with error handling
- Test thoroughly before committing

### Documentation
- Document all public functions
- Include usage examples
- Update system documentation for new features
- Maintain accuracy with code changes

### Version Control
- Use descriptive commit messages
- Create branches for major features
- Review code before merging
- Tag releases appropriately

## Version History

### Current Version
- Enhanced event system with built-in events
- Improved database persistence
- Expanded UI framework
- Performance optimizations

### Planned Features
- Enhanced AI behavior scripting
- Improved networking utilities
- Extended database functionality
- Additional UI components

## Support

### Documentation
- System documentation in `docs/systems/`
- Function documentation in source files
- Usage examples throughout codebase

### Community
- GitHub issues for bug reports
- Discussion forums for questions
- Community contributions welcome

## License

Paradigm is released under the same license as the parent project. See LICENSE.txt for details.
