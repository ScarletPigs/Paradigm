# Scheduler and Jobs System

## Overview

The Paradigm Scheduler System provides a lightweight, centralized job scheduling and execution framework for Arma 3 missions. It manages repetitive, frequent tasks that need to run at regular intervals without the overhead of spawning separate scripts. The scheduler is designed for high-frequency, lightweight operations that should not block mission execution.

## Execution Context

**The scheduler runs independently on each machine** - both server and clients maintain their own scheduler instance:

- **Server**: Initialized in `para_server_init.sqf` - handles server-side jobs (AI management, cleanup, vehicle tracking, etc.)
- **Clients**: Initialized in `para_player_init_client.sqf` - handles client-side jobs (UI updates, local monitoring, player-specific tasks)

Each scheduler instance is completely independent, allowing for:
- **Server-side jobs**: Mission logic, AI behavior, world state management
- **Client-side jobs**: User interface updates, local performance monitoring, player-specific features
- **Network coordination**: Jobs can synchronize data between server and clients when needed

**Important**: Jobs are not automatically synchronized between machines. If you need cross-network coordination, you must explicitly handle it within your job code using `remoteExec`, `publicVariable`, or the event system.

## Core Concepts

### Jobs
Jobs are recurring tasks that execute at specified intervals. Each job consists of:
- **Code** - The function or code block to execute
- **Parameters** - Data passed to the job code on each execution
- **Tick Delay** - Minimum time between executions (in seconds)
- **Iterations** - Number of times to run (default: infinite)
- **Job ID** - Unique identifier for job management

### Scheduler Loop
The scheduler runs as a single background script that:
1. Checks each registered job's timing
2. Executes jobs that are ready to run
3. Manages job lifecycle (start, run, cleanup)
4. Handles job removal and resource cleanup

### Design Philosophy
- **Lightweight** - Jobs should be fast, non-blocking operations
- **Centralized** - Single scheduler manages all recurring tasks
- **Reliable** - Built-in monitoring and automatic restart capability
- **Resource Efficient** - Automatic cleanup of completed jobs

## API Reference

### Core Functions

#### `para_g_fnc_scheduler_add_job`
Registers a new job with the scheduler.

**Parameters:**
- `_jobId` [String] - Unique identifier for the job
- `_code` [Code] - Function or code block to execute
- `_parameters` [Array] - Parameters to pass to the job code
- `_tickDelay` [Number] - Minimum delay between executions (seconds)
- `_iterationsToRun` [Number, Optional] - Number of times to run (-1 = infinite, default)

**Returns:**
- `_job` [Location] - Job namespace for advanced control

**Example:**
```sqf
["cleanup_job", {
    params ["_maxItems"];
    // Cleanup old items if over limit
    if (count allItems > _maxItems) then {
        { deleteVehicle _x } forEach (allItems select [0, 10]);
    };
}, [1000], 30] call para_g_fnc_scheduler_add_job;
```

#### `para_g_fnc_scheduler_remove_job`
Removes a job from the scheduler (after next execution).

**Parameters:**
- `_jobId` [String] - ID of the job to remove

**Returns:**
- `_success` [Boolean] - Whether job was found and marked for removal

**Example:**
```sqf
["cleanup_job"] call para_g_fnc_scheduler_remove_job;
```

#### `para_g_fnc_scheduler_get_job`
Retrieves a job namespace by ID for advanced manipulation.

**Parameters:**
- `_jobId` [String] - ID of the job to retrieve

**Returns:**
- `_job` [Location] - Job namespace or locationNull if not found

**Example:**
```sqf
private _job = ["my_job"] call para_g_fnc_scheduler_get_job;
if (!isNull _job) then {
    _job setVariable ["remainingIterations", 5]; // Run 5 more times
};
```

#### `para_g_fnc_scheduler_subsystem_init`
Initializes the scheduler subsystem. Called automatically during mission initialization.

**Execution Context:** Runs on both server and clients independently

**Parameters:** None

**Returns:** Nothing

**Automatic Initialization:**
- **Server**: Called in `para_server_init.sqf`
- **Clients**: Called in `para_player_init_client.sqf`

**Example:**
```sqf
// Manual initialization (usually not needed)
call para_g_fnc_scheduler_subsystem_init;
```

### Advanced Job Control

#### Setting Limited Iterations
Jobs can be configured to run a specific number of times:

```sqf
// Run exactly 10 times
["limited_job", {
    hint "This will run 10 times";
}, [], 5, 10] call para_g_fnc_scheduler_add_job;
```

#### One-Time Execution
For delayed execution without repetition:

```sqf
// Run once after 30 seconds
["delayed_action", {
    hint "Executed after delay";
}, [], 30, 1] call para_g_fnc_scheduler_add_job;
```

#### Dynamic Job Modification
Jobs can modify their own execution parameters:

```sqf
["adaptive_job", {
    private _job = ["adaptive_job"] call para_g_fnc_scheduler_get_job;
    
    // Change delay based on conditions
    if (count allPlayers > 10) then {
        _job setVariable ["tickDelay", 1]; // More frequent
    } else {
        _job setVariable ["tickDelay", 5]; // Less frequent
    };
}, [], 5] call para_g_fnc_scheduler_add_job;
```

## Built-in System Jobs

The following jobs are automatically registered by various Paradigm subsystems:

### Server-Side Jobs

#### Event Dispatcher
Processes queued events from the event system.

**Job ID:** `event_dispatcher`
**Frequency:** Every 1 second
**Execution:** Server and Clients (each processes their own event queue)
**Purpose:** Execute event handlers for dispatched events

```sqf
["event_dispatcher", para_g_fnc_event_dispatcher_job, [], 1] call para_g_fnc_scheduler_add_job;
```

#### AI Behavior Manager
Manages AI group behaviors and reactions.

**Job ID:** `behaviour_manager`
**Frequency:** Every 3 seconds
**Execution:** Global (both server and clients)
**Purpose:** Update AI behaviors for all managed groups

```sqf
["behaviour_manager", para_g_fnc_ai_run_behaviours_all_groups, [], 3] call para_g_fnc_scheduler_add_job;
```

#### Cleanup System
Manages automatic cleanup of objects and bodies.

**Job ID:** `cleanup`
**Frequency:** Every 5 seconds
**Execution:** Server only
**Purpose:** Remove old objects, bodies, and debris

```sqf
["cleanup", {call para_s_fnc_cleanup_job}, [], 5] call para_g_fnc_scheduler_add_job;
```

#### Day/Night Cycle
Manages time-based mission events and lighting.

**Job ID:** `day_night_cycle`
**Frequency:** Every 120 seconds (2 minutes)
**Execution:** Server only
**Purpose:** Handle dawn/dusk events and time progression

```sqf
["day_night_cycle", para_s_fnc_day_night_job, [], 120] call para_g_fnc_scheduler_add_job;
```

#### Vehicle Asset Manager
Monitors and manages vehicle spawn points and states.

**Job ID:** `vehicle_asset_manager`
**Frequency:** Every 5 seconds
**Execution:** Server only
**Purpose:** Track vehicle status and handle respawning

```sqf
["vehicle_asset_manager", vn_mf_fnc_veh_asset_job, [], 5] call para_g_fnc_scheduler_add_job;
```

#### Load Balancer
Distributes AI processing load across clients.

**Job ID:** `loadbal_fps_aggregator`
**Frequency:** Every 10 seconds
**Execution:** Server only
**Purpose:** Monitor performance and balance AI distribution

```sqf
["loadbal_fps_aggregator", {call para_s_fnc_loadbal_fps_aggregator}, [], 10] call para_g_fnc_scheduler_add_job;
```

### Client-Side Jobs

#### Info Panel Updates
Updates the client's information panel display.

**Job ID:** Various UI-related jobs
**Frequency:** 1-5 seconds typically
**Execution:** Client only
**Purpose:** Refresh UI elements, player stats, local information

## Usage Patterns

### Subsystem Management
Most systems register jobs during initialization:

```sqf
// In subsystem init function
["my_subsystem_manager", my_subsystem_fnc_job, [], 15] call para_g_fnc_scheduler_add_job;
```

### Periodic Monitoring
Monitor conditions and trigger actions:

```sqf
["zone_monitor", {
    params ["_zoneMarker"];
    private _playersInZone = allPlayers select {_x inArea _zoneMarker};
    
    if (count _playersInZone > 0) then {
        ["playersEnteredZone", [_zoneMarker, _playersInZone]] call para_g_fnc_event_dispatch;
    };
}, ["zone_alpha"], 10] call para_g_fnc_scheduler_add_job;
```

### State Machine Management
Drive state machines with regular updates:

```sqf
["state_machine", {
    params ["_stateMachine"];
    
    // Process current state
    private _currentState = _stateMachine getVariable "currentState";
    [_stateMachine] call (_stateMachine getVariable _currentState);
    
    // Check for state transitions
    [_stateMachine] call (_stateMachine getVariable "checkTransitions");
}, [myStateMachine], 2] call para_g_fnc_scheduler_add_job;
```

### Resource Management
Handle periodic cleanup and maintenance:

```sqf
["memory_cleanup", {
    // Clean up old data structures
    para_old_data = para_old_data select {!isNull _x};
    
    // Garbage collection hint
    if (count para_old_data == 0) then {
        call para_g_fnc_garbage_collect;
    };
}, [], 60] call para_g_fnc_scheduler_add_job;
```

### Conditional Job Removal
Jobs can remove themselves based on conditions:

```sqf
["mission_phase_monitor", {
    if (missionNamespace getVariable ["missionPhase", ""] == "COMPLETED") then {
        ["mission_phase_monitor"] call para_g_fnc_scheduler_remove_job;
        hint "Mission completed, stopping monitor";
    };
}, [], 5] call para_g_fnc_scheduler_add_job;
```

## Advanced Usage

### Job Networking
Jobs can handle network synchronization between server and clients:

```sqf
// Server-side data broadcaster
["server_data_sync", {
    if (isServer) then {
        // Server collects and broadcasts data
        private _syncData = [time, count allPlayers, count allUnits];
        missionNamespace setVariable ["globalSyncData", _syncData, true];
    };
}, [], 10] call para_g_fnc_scheduler_add_job;

// Client-side data processor  
["client_data_sync", {
    if (!isServer) then {
        // Clients process server data
        private _syncData = missionNamespace getVariable ["globalSyncData", []];
        if (!(_syncData isEqualTo [])) then {
            // Update local UI or systems with server data
            [_syncData] call update_local_systems;
        };
    };
}, [], 5] call para_g_fnc_scheduler_add_job;
```

### Machine-Specific Jobs
Design jobs to run only where appropriate:

```sqf
// Server-only job for mission logic
if (isServer) then {
    ["mission_state_manager", {
        // Server handles mission state
        call check_mission_objectives;
        call update_mission_progress;
    }, [], 30] call para_g_fnc_scheduler_add_job;
};

// Client-only job for UI updates
if (hasInterface) then {
    ["ui_refresh", {
        // Update player's UI elements
        call refresh_stamina_bar;
        call update_minimap_markers;
    }, [], 1] call para_g_fnc_scheduler_add_job;
};

// Headless client job for AI processing
if (!hasInterface && !isServer) then {
    ["hc_ai_manager", {
        // Headless client handles AI
        call process_ai_groups;
    }, [], 5] call para_g_fnc_scheduler_add_job;
};
```

### Performance Monitoring
Monitor and adapt job performance:

```sqf
["performance_monitor", {
    private _startTime = diag_tickTime;
    
    // Perform work
    call my_expensive_function;
    
    private _executionTime = diag_tickTime - _startTime;
    if (_executionTime > 0.1) then {
        ["WARNING: Scheduler job taking too long: %1ms", _executionTime * 1000] call BIS_fnc_logFormat;
    };
}, [], 15] call para_g_fnc_scheduler_add_job;
```

### Dynamic Job Creation
Create jobs based on runtime conditions:

```sqf
// Create temporary jobs for special events
if (missionNamespace getVariable ["specialEventActive", false]) then {
    ["special_event_handler", {
        // Handle special event logic
        if (!(missionNamespace getVariable ["specialEventActive", false])) then {
            ["special_event_handler"] call para_g_fnc_scheduler_remove_job;
        };
    }, [], 1] call para_g_fnc_scheduler_add_job;
};
```

## Job Development Guidelines

### Performance Considerations

#### Keep Jobs Lightweight
- Avoid heavy computations in scheduler jobs
- Use spawn for long-running operations
- Limit processing time to < 10ms per execution

```sqf
// Good: Lightweight check
["player_count_monitor", {
    if (count allPlayers != (missionNamespace getVariable ["lastPlayerCount", 0])) then {
        missionNamespace setVariable ["lastPlayerCount", count allPlayers];
        ["playerCountChanged", [count allPlayers]] call para_g_fnc_event_dispatch;
    };
}, [], 5] call para_g_fnc_scheduler_add_job;

// Bad: Heavy computation
["bad_performance_job", {
    // This blocks the scheduler!
    {
        private _nearbyEnemies = _x nearEntities [["Man"], 500];
        // Heavy processing for each unit...
    } forEach allUnits;
}, [], 1] call para_g_fnc_scheduler_add_job;
```

#### Use Appropriate Intervals
- Match frequency to actual need
- Don't run checks more often than necessary
- Consider using events instead of polling

```sqf
// Good: Reasonable interval for file I/O
["save_progress", {
    ["SAVE", "playerProgress", player getVariable ["progress", []]] call para_s_fnc_profile_db;
}, [], 300] call para_g_fnc_scheduler_add_job; // Every 5 minutes

// Bad: Too frequent for expensive operation
["bad_frequency", {
    // Database operation every second!
    ["GET", "allPlayerData", []] call para_s_fnc_profile_db;
}, [], 1] call para_g_fnc_scheduler_add_job;
```

### Error Handling
Protect scheduler stability with error handling:

```sqf
["robust_job", {
    params ["_data"];
    
    try {
        // Job logic that might fail
        [_data] call risky_function;
    } catch {
        ["Job 'robust_job' error: %1", _exception] call BIS_fnc_logFormat;
    };
}, [someData], 10] call para_g_fnc_scheduler_add_job;
```

### Resource Management
Clean up resources when jobs complete:

```sqf
["cleanup_aware_job", {
    params ["_trackedObjects"];
    
    // Clean up null references
    _trackedObjects = _trackedObjects select {!isNull _x};
    
    // Update parameters for next run
    private _job = ["cleanup_aware_job"] call para_g_fnc_scheduler_get_job;
    _job setVariable ["parameters", [_trackedObjects]];
    
    // Remove job if no more objects to track
    if (count _trackedObjects == 0) then {
        ["cleanup_aware_job"] call para_g_fnc_scheduler_remove_job;
    };
}, [initialObjects], 30] call para_g_fnc_scheduler_add_job;
```

## Debugging and Monitoring

### Enable Debug Logging
Set the debug flag to see scheduler activity:

```sqf
debugScheduler = true; // Enable scheduler logging
```

### Job Performance Monitoring
The scheduler automatically tracks job performance for profiling:

```sqf
// Job names appear in the Arma 3 profiler
// Use descriptive job IDs for easier debugging
["descriptive_job_name", myCode, [], 10] call para_g_fnc_scheduler_add_job;
```

### Common Issues

#### Job Not Running
- Check job ID is unique
- Verify tick delay is reasonable
- Ensure scheduler is initialized

#### Performance Problems
- Jobs taking too long (> 50ms)
- Too many jobs running simultaneously
- Recursive job creation causing memory leaks

#### Memory Leaks
- Jobs not cleaning up resources
- Accumulating data in job parameters
- Failed job removal on mission end

## Implementation Details

### Scheduler Architecture
The scheduler uses a single `spawn`ed script that:
1. Loops continuously with 0.1 second sleep
2. Checks each job's timing against `diag_tickTime`
3. Executes ready jobs with their parameters
4. Manages job lifecycle and cleanup

### Storage Format
Jobs are stored as an array of `[jobId, jobNamespace]` pairs where the namespace contains:
- `code` - The function to execute
- `parameters` - Parameters array
- `tickDelay` - Seconds between executions
- `lastTickTime` - When the job last ran
- `remainingIterations` - Executions remaining (-1 = infinite)
- `startTime` - When the job was created
- `removeFromScheduler` - Flag for job removal

### Automatic Restart
A monitor script watches the scheduler and restarts it if it crashes:

```sqf
// Monitor runs separately to ensure reliability
0 spawn para_g_fnc_scheduler_monitor;
```

## Integration with Other Systems

### Event System Integration
The scheduler drives the event system through the event dispatcher job:

```sqf
["event_dispatcher", para_g_fnc_event_dispatcher_job, [], 1] call para_g_fnc_scheduler_add_job;
```

### Task System Integration
Tasks are executed as scheduler jobs:

```sqf
[_taskFrameworkId, _taskScript, [_taskDataStore], 5] call para_g_fnc_scheduler_add_job;
```

### Subsystem Coordination
Most Paradigm subsystems use scheduler jobs for their main loops:
- AI Management
- Cleanup System
- Vehicle Management
- Load Balancing
- Day/Night Cycle

## Best Practices

### Job Design
1. **Single Responsibility** - Each job should have one clear purpose
2. **Stateless When Possible** - Avoid relying on external state
3. **Graceful Degradation** - Handle missing data gracefully
4. **Self-Monitoring** - Jobs should detect their own completion conditions

### Naming Conventions
- Use descriptive, unique job IDs
- Include subsystem prefix: `"ai_behavior_manager"`
- Use underscores for readability: `"zone_capture_monitor"`

### Lifecycle Management
- Initialize jobs in subsystem init functions
- Remove jobs during subsystem shutdown
- Use limited iterations for temporary jobs
- Monitor job performance and adjust intervals

### Performance Guidelines
- Target < 5ms execution time per job
- Use appropriate tick delays (don't over-poll)
- Batch operations when possible
- Offload heavy work to spawned scripts

The Paradigm Scheduler System provides a robust foundation for managing recurring tasks in Arma 3 missions. By following these patterns and guidelines, you can create efficient, maintainable systems that enhance mission functionality without impacting performance.
