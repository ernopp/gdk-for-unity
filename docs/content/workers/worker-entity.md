
**Warning:** The [alpha](https://docs.improbable.io/reference/latest/shared/release-policy#maturity-stages) release is for evaluation purposes only.

-----
[//]: # (Doc of docs reference 15.2a)

# Workers: Worker entity
_This document relates to both *[GameObject-MonoBehaviour and  ECS workflows](../intro-workflows-spos-entities.md)_


Before reading this document, see the documentation on [workers in the GDK](./workers-in-the-gdk.md).

Each of the workers in your project must have exactly one [ECS entity](../glossary.md#unity-ecs-entity) in its [worker-ECS world](./workers-in-the-gdk.md# workers-and-ecs-worlds) at any point in time. To uniquely identify the worker entity of your current worker, the worker entity has the `WorkerEntityTag` component attached to it.

The worker’s worker entity performs certain tasks:


  * send and receive [commands](../glossary.md#commands) before the worker has checked out any SpatialOS entities.
  * register changes to the state of the Runtime connection (that is whether the worker is connected to the [Runtime](../glossary.md#spatialos-runtime) or not) by filtering for the following [temporary components](temporary-components.md):
 	* `OnConnected`: the worker just connected to the SpatialOS [Runtime](../glossary.md#spatialos-runtime).
 	* `OnDisconnected`: the worker just disconnected from the SpatialOS [Runtime](../glossary.md#spatialos-runtime). This is an `ISharedComponentData` and stores the reason for the disconnection as a `string`.




## How to run logic when the worker has just connected

You can use the worker to check in an ECS system to see whether the worker just
connected. This allows you to handle any initialization logic necessary.

**Example**<br/>
```csharp
using Improbable.Gdk.Core;
using Unity.Entities;
using UnityEngine;

public class HandleConnectSystem : ComponentSystem
{
	private struct Data
	{
    	public readonly int Length;
    	[ReadOnly] public ComponentDataArray<OnConnected> OnConnected;
    	[ReadOnly] public ComponentDataArray<WorkerEntityTag> DenotesWorkerEntity;
	}

	[Inject] private Data data;

	protected override void OnUpdate()
	{
    	Debug.Log("Worker just connected!");
	}
}
```

## How to run logic when the worker has just disconnected
You can use the worker to check in an ECS system to see whether the worker just disconnected. This allows you to handle any clean-up logic necessary.

**Example**<br/>
```csharp
using Improbable.Gdk.Core;
using Unity.Entities;
using UnityEngine;

public class HandleDisconnectSystem : ComponentSystem
{
	private struct Data
	{
    	public readonly int Length;
    	[ReadOnly] public SharedComponentDataArray<OnDisconnected> OnDisconnected;
    	[ReadOnly] public ComponentDataArray<WorkerEntityTag> DenotesWorkerEntity;
	}

	[Inject] private Data data;

	protected override void OnUpdate()
	{
    	var reasonForDisconnect = data.OnDisconnected[0].ReasonForDisconnect;
    	Debug.Log($"Got disconnected: {reasonForDisconnect}");
	}
}
```

## How to send a command using the worker entity
The worker entity has all [command sender components](../ecs/commands.md) attached to it.
By filtering for these components, you are able to send commands even if you don't have any [SpatialOS entities](../glossary.md#spatialos-entity) which is [checked out](../glossary.md#checking-out).

```csharp
public class CreateCreatureSystem : ComponentSystem
{
	private struct Data
	{
    	public readonly int Length;
    	[ReadOnly] public ComponentDataArray<WorkerEntityTag> DenotesWorkerEntity;
    	public ComponentDataArray<WorldCommands.CreateEntity.CommandSender> CreateEntitySender;
	}

	[Inject] private Data data;

	protected override void OnUpdate()
	{
    	var requestSender = data.CreateEntitySender[0];
    	var entity = CreatureTemplate.CreateCreatureEntityTemplate(new Coordinates(0, 0, 0));
    	requestSender.RequestsToSend.Add(WorldCommands.CreateEntity.CreateRequest
    	(
        	entity
    	));
    	data.CreateEntitySender[0] = requestSender;
	}
}
```
