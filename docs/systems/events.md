# Event System

## Overview

The Paradigm Event System provides a lightweight, flexible event-driven architecture for Arma 3 missions. It enables decoupled communication between different systems and supports dynamic event handling without tight coupling between components.

## Core Concepts

### Events
Events are named occurrences that can carry data and trigger registered handlers. Events follow a publish-subscribe pattern where:
- **Publishers** dispatch events when something happens
- **Subscribers** register handlers to respond to specific events
- **Data** can be passed from publishers to subscribers

### Event Flow
```
Event Occurs → Dispatch Event → Find Handlers → Execute Handlers → Continue Execution
```

## API Reference

### Core Functions

#### `para_g_fnc_event_add_handler`
Registers a handler for a specific event type.

**Parameters:**
- `_eventName` [String] - Name of the event to listen for
- `_handler` [Array] - Handler definition array containing:
  - `_handlerCode` [Code] - Code to execute when event fires
  - `_handlerParams` [Array] - Parameters to pass to the handler

**Returns:** 
- `_handler` [Array] - The registered handler

**Example:**
```sqf
["playerKilled", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_unit", "_killer"];
    hint format ["%1 was killed by %2", name _unit, name _killer];
}, []]] call para_g_fnc_event_add_handler;
```

#### `para_g_fnc_event_dispatch`
Triggers an event and executes all registered handlers.

**Parameters:**
- `_eventName` [String] - Name of the event to dispatch
- `_args` [Any] - Data to pass to event handlers

**Returns:** Nothing

**Example:**
```sqf
["playerJoined", [player, playerSide]] call para_g_fnc_event_dispatch;
```

### Handler Format
Event handlers receive parameters in this format:
```sqf
params ["_handlerParams", "_eventParams"];
```
Where:
- `_handlerParams` - The handler-specific parameters array provided during registration
- `_eventParams` - The event data passed when the event is dispatched

The handler code then extracts individual parameters:
```sqf
_handlerParams params []; // Extract handler parameters if any
_eventParams params ["_param1", "_param2", ...]; // Extract event parameters
```

## Built-in Events

### Vehicle Events

#### `vehicleCreated`
Fired when a new vehicle is detected in the mission.

**Parameters:**
- `_vehicle` [Object] - The created vehicle

**Example:**
```sqf
["vehicleCreated", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_vehicle"];
    diag_log format ["New vehicle: %1", typeOf _vehicle];
}, []]] call para_g_fnc_event_add_handler;
```

**Dispatch:**
```sqf
["vehicleCreated", [_newVehicle]] call para_g_fnc_event_dispatch;
```

### Task Events

#### `taskCreated`
Fired when a new task is created in the mission.

**Parameters:**
- `_taskId` [String] - Unique task identifier
- `_taskData` [Array] - Task configuration data

**Example:**
```sqf
["taskCreated", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_taskId", "_taskData"];
    systemChat format ["New task: %1", _taskId];
}, []]] call para_g_fnc_event_add_handler;
```

#### `taskCompleted`
Fired when a task is completed by players.

**Parameters:**
- `_taskId` [String] - Completed task identifier
- `_completionData` [Array] - Task completion details

**Example:**
```sqf
["taskCompleted", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_taskId", "_completionData"];
    ["TaskCompleted", [_taskId]] call para_g_fnc_event_dispatch;
}, []]] call para_g_fnc_event_add_handler;
```

### Player Events

#### `playerConnected`
Fired when a player connects to the server.

**Parameters:**
- `_player` [Object] - The connecting player unit
- `_playerInfo` [Array] - Connection details

**Example:**
```sqf
["playerConnected", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_player", "_playerInfo"];
    diag_log format ["Player connected: %1", name _player];
}, []]] call para_g_fnc_event_add_handler;
```

#### `playerDisconnected`
Fired when a player disconnects from the server.

**Parameters:**
- `_playerId` [String] - Player's unique ID
- `_playerName` [String] - Player's display name

### Zone Events

#### `zoneOpened`
Fired when a new zone becomes available for operations.

**Parameters:**
- `_zoneName` [String] - Name of the opened zone
- `_zoneData` [Array] - Zone configuration data

**Example:**
```sqf
["zoneOpened", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName", "_zoneData"];
    systemChat format ["Zone %1 is now open!", _zoneName];
}, []]] call para_g_fnc_event_add_handler;
```

#### `zoneCompleted`
Fired when a zone is successfully captured by players.

**Parameters:**
- `_zoneName` [String] - Name of the captured zone
- `_captureData` [Array] - Capture completion details

#### `zoneActivated`
Fired when a zone becomes populated with AI objectives (becomes contested).

**Parameters:**
- `_zoneName` [String] - Name of the zone that became active

**Example:**
```sqf
["zoneActivated", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName"];
    systemChat format ["Zone %1 is now contested!", _zoneName];
}, []]] call para_g_fnc_event_add_handler;
```

**Dispatch:**
```sqf
["zoneActivated", [_zoneName]] call para_g_fnc_event_dispatch;
```

#### `zoneDeactivated`
Fired when a zone cleans all AI objectives (no longer contested).

**Parameters:**
- `_zoneName` [String] - Name of the zone that became inactive

**Example:**
```sqf
["zoneDeactivated", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName"];
    systemChat format ["Zone %1 is no longer contested", _zoneName];
}, []]] call para_g_fnc_event_add_handler;
```

**Dispatch:**
```sqf
["zoneDeactivated", [_zoneName]] call para_g_fnc_event_dispatch;
```

#### Zone Event Types
The zone system distinguishes between different types of zone states:

- **`zoneOpened`** - Zone becomes available for tasks (director system)
- **`zoneCompleted`** - Zone is captured/secured by players (director system)  
- **`zoneActivated`** - Zone gains AI objectives and becomes contested (AI system)
- **`zoneDeactivated`** - Zone loses all AI objectives, no longer contested (AI system)

A zone can be "opened" by the director but not "activated" if no AI objectives are spawned yet. Conversely, a zone can lose its AI objectives (become "deactivated") but still remain "opened" by the director system.

### Building Feature Events

#### `buildingFeatureActivated`
Fired when a building feature (like respawn point) is activated.

**Parameters:**
- `_building` [Object] - The building object
- `_featureType` [String] - Type of feature activated

**Example:**
```sqf
["buildingFeatureActivated", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_building", "_featureType"];
    if (_featureType == "respawn") then {
        systemChat "New respawn point available!";
    };
}, []]] call para_g_fnc_event_add_handler;
```

### Custom Events

#### `missionPhaseChanged`
Fired when the mission progresses to a new phase.

**Parameters:**
- `_newPhase` [String] - Name of the new mission phase
- `_phaseData` [Array] - Phase-specific data

#### `suppliesDelivered`
Fired when supply drops are delivered to a location.

**Parameters:**
- `_location` [Array] - Delivery position
- `_supplies` [Array] - List of delivered items

## Usage Patterns

### Event-Driven System Communication

Instead of direct function calls between systems, use events for loose coupling:

**Bad:**
```sqf
// Direct coupling - hard to maintain
vn_mf_fnc_zone_capture_complete call vn_mf_fnc_update_task_status;
```

**Good:**
```sqf
// Event-driven - flexible and extensible
["zoneCaptured", [_zoneName, _captureData]] call para_g_fnc_event_dispatch;
```

### Multiple Handler Registration

Multiple systems can respond to the same event:

```sqf
// Task system responds to zone capture
["zoneCaptured", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName", "_captureData"];
    // Update task status
}, []]] call para_g_fnc_event_add_handler;

// Statistics system responds to zone capture
["zoneCaptured", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName", "_captureData"];
    // Record statistics
}, []]] call para_g_fnc_event_add_handler;

// Notification system responds to zone capture
["zoneCaptured", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName", "_captureData"];
    // Show notification
}, []]] call para_g_fnc_event_add_handler;
```

### Conditional Event Handling

Use conditions within handlers for selective processing:

```sqf
["vehicleCreated", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_vehicle"];
    
    // Only process aircraft
    if (_vehicle isKindOf "Air") then {
        [_vehicle] call my_fnc_setup_aircraft;
    };
}, []]] call para_g_fnc_event_add_handler;
```

### Data Transformation

Transform event data for specific use cases:

```sqf
["taskCompleted", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_taskId", "_completionData"];
    
    // Transform for statistics system
    private _statsData = [_taskId, time, _completionData];
    ["statisticsUpdate", [_statsData]] call para_g_fnc_event_dispatch;
}, []]] call para_g_fnc_event_add_handler;
```

## Advanced Usage

### Event Chaining

Events can trigger other events for complex workflows:

```sqf
["zoneCaptured", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName", "_captureData"];
    
    // Trigger related events
    ["taskCompleted", [format["capture_%1", _zoneName]]] call para_g_fnc_event_dispatch;
    ["areaSecured", [_zoneName]] call para_g_fnc_event_dispatch;
}, []]] call para_g_fnc_event_add_handler;
```

### Error Handling in Events

Protect against handler failures:

```sqf
["criticalEvent", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_data"];
    
    try {
        // Handler logic
        [_data] call risky_function;
    } catch {
        diag_log format ["Event handler error: %1", _exception];
    };
}, []]] call para_g_fnc_event_add_handler;
```

### Dynamic Handler Registration

Register handlers based on runtime conditions:

```sqf
if (isServer) then {
    ["playerConnected", [{
        // Server-only handling
    }, []]] call para_g_fnc_event_add_handler;
};

if (hasInterface) then {
    ["notificationReceived", [{
        // Client-only handling
    }, []]] call para_g_fnc_event_add_handler;
};
```

## Performance Considerations

### Handler Efficiency
- Keep handlers lightweight and fast
- Avoid heavy computation in event handlers
- Use separate threads for long-running operations

```sqf
["heavyProcessingEvent", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_data"];
    
    // Spawn separate thread for heavy work
    _data spawn {
        // Heavy processing here
    };
}, []]] call para_g_fnc_event_add_handler;
```

### Event Frequency
- Be mindful of high-frequency events
- Consider batching or throttling for performance-critical events
- Use appropriate event granularity

### Memory Management
- Event handlers persist for the mission duration
- No built-in handler removal (by design for simplicity)
- Structure handlers to be stateless when possible

## Implementation Details

### Storage Mechanism
Handlers are stored in mission namespace variables using this pattern:
```sqf
para_l_eventHandlers_<eventName>
```

### Execution Order
- Handlers execute in registration order
- No guaranteed execution order between handlers
- Design handlers to be order-independent

### Scope and Locality
- Events are local to the machine where dispatched
- Cross-network event propagation requires explicit `remoteExec`
- Global events need manual synchronization

## Best Practices

### Event Naming
- Use descriptive, consistent naming: `playerKilled`, `vehicleCreated`
- Include context when needed: `zoneContested`, `taskCompleted`
- Avoid abbreviations: use `vehicleDestroyed` not `vehDest`

### Handler Design
- Keep handlers focused and single-purpose
- Avoid side effects when possible
- Document handler behavior and expectations

### Error Prevention
- Validate parameters in handlers
- Use defensive programming practices
- Handle edge cases gracefully

### Documentation
- Document custom events and their parameters
- Provide usage examples
- Maintain event documentation as systems evolve

## Integration Examples

### Vehicle Saving System Integration
```sqf
["vehicleCreated", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_vehicle"];
    // Add to vehicle tracking if friendly
    if ([_vehicle] call vn_mf_fnc_is_friendly_vehicle) then {
        [_vehicle] call vn_mf_fnc_add_vehicle_to_tracking;
    };
}, []]] call para_g_fnc_event_add_handler;
```

### Task System Integration
```sqf
["zoneCaptured", [{
    params ["_handlerParams", "_eventParams"];
    _handlerParams params [];
    _eventParams params ["_zoneName", "_captureData"];
    // Complete related tasks
    {
        if (_x select 1 == _zoneName) then {
            [_x select 0] call vn_mf_fnc_task_complete;
        };
    } forEach vn_mf_active_capture_tasks;
}, []]] call para_g_fnc_event_add_handler;
```

## Related Systems

- **Vehicle Saving System** - Uses `vehicleCreated` events for automatic tracking
- **Task System** - Dispatches task-related events for mission progression
- **Zone System** - Provides zone state change events
- **Building Features** - Dispatches activation events for base building
