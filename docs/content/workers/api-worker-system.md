**Warning:** The [alpha](https://docs.improbable.io/reference/latest/shared/release-policy#maturity-stages) release is for evaluation purposes only.

-----
[//]: # (Doc of docs reference 15.2)

#  Workers: API - Workers system

_This document relates to both [GameObject-MonoBehaviour and ECS workflows](../intro-workflows-spos-entities.md)._

See first the documentation on [Workers in the GDK](./workers-in-the-gdk.md) and the [Worker API](./api-worker.md)

The `WorkerSystem` class stores information about a worker during [Runtime](../glossary.md#spatialos-runtime). You can use `WorkerSystem` to access information about the worker during Runtime; any system running in the same [ECS world]( as a worker can access the `WorkerSystem` class.

** Fields **

| Field         	| Type               	| Description                	|
|-------------------|------------------------|--------------------------------|
| Connection	| [Connection](../connecting-to-spos.md) | The connection to the SpatialOS Runtime. You can use it to send data and messages. |
| WorkerType	| string             	| The [type of this worker](../glossary.md#type-of-worker). |
| Origin    	| Vector3            	| The vector by which we [translate] (TODO: link to glossary - translation/translate) all ECS entities added to a worker. This is useful when running multiple workers in the same scene. You can choose to set a worker origin to be large enough so that entities that are visible to or checked out by different workers don’t interact with each other. |
| WorkerEntity  | Entity             	| The corresponding (Worker entity)[TODO: Add link to GDocs 15.2.a] which allows you to query the current state of the worker as well as send and receive commands. |
| LogDispatcher | ILogDispatcher     	| A reference to the [logger](ecs/logging.md) that you can use to log to the Unity Editor’s console and the [SpatialOS Console](../glossary.md#console) |

** Methods **

```csharp
bool TryGetEntity(EntityId entityId, out Entity entity);
```
Parameters:
  * `EntityId entityId`: The entity ID of the SpatialOS entity that you want to retrieve.
  * `Entity entity`: The ECS entity that represents the SpatialOS entity that you want to retrieve. It will be `Entity.Null`, if this ECS entity does not exist.

Returns: true, if the queried SpatialOS entity is checked out on this worker, false otherwise.

### ECS: How to access the WorkerSystem in the ECS workflow

You can inject the worker system in any `ComponentSystem` that is in the same [ECS world](../glossary.md#unity-ecs-world) as the `WorkerSystem`.
See the code example below for a guide on how to inject it:

**Example**</br>

```csharp
using Improbable.Gdk.Core;
using Unity.Entities;
using UnityEngine;

public class YourSystem : ComponentSystem
{
	[Inject] WorkerSystem Worker;

	protected override void OnCreateManager()
	{
    	base.OnCreateManager();
    	// The injected WorkerSystem will already be available here
	}

	protected override void OnUpdate()
{
    	// Do something
	}
}
```

## GameObject: How to access the WorkerSystem in the GameObject and MonoBehaviour workflow

```csharp
public class CubeSpawnerInputBehaviour : MonoBehaviour
{

  private void OnEnable()
  {
    var spatialOSComponent = GetComponent<SpatialOSComponent>();
    var worker = spatialOSComponent.Worker;
    // do something with your worker
  }
}
```

