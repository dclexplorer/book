# Cinematic Camera
This experimental feature enables you to run a camera attached to an arbitrary entity. 

## What it changes
This proposal changed the `CameraModeArea` and `AvatarModifierArea` components.

### CameraModeArea

<details>
  <summary>Difference between experimental version and original one</summary>

```diff
declare const enum CameraType {
    CT_FIRST_PERSON = 0,
    CT_THIRD_PERSON = 1,
+    /** CT_CINEMATIC - controlled by the scene */
+    CT_CINEMATIC = 2
}

interface PBCameraModeArea {
    /** the 3D size of the region */
    area: Vector3 | undefined;
    /** the camera mode to enforce */
    mode: CameraType;
+    cinematicSettings?: CinematicSettings | undefined;
+    /** if true, the player will be considered inside the area when they are within 0.3m of the area. default true */
+    useColliderRange?: boolean | undefined;
}

+interface CinematicSettings {
+    /** Entity that defines the cinematic camera transform. */
+    cameraEntity: number;
+    /**
+     * Position -> camera's position
+     * Rotation -> camera's direction
+     * scale.z -> zoom level
+     * scale.x and scale.y -> unused
+     */
+    allowManualRotation?: boolean | undefined;
+    /** how far the camera can rotate around the y-axis / look left/right, in radians. default unrestricted */
+    yawRange?: number | undefined;
+    /** how far the camera can rotate around the x-axis / look up-down, in radians. default unrestricted */
+    pitchRange?: number | undefined;
+    /** note: cameras can never look up/down further than Vec3::Y */
+    rollRange?: number | undefined;
+    /** minimum zoom level. must be greater than 0. defaults to the input zoom level */
+    zoomMin?: number | undefined;
+    /** maximum zoom level. must be greater than 0. defaults to the input zoom level */
+    zoomMax?: number | undefined;
+}
```

</details>

Summary: 
- It adds `CT_CINAMATIC` option to `CameraType` enum which is used for the property `mode` in the `CameraModeArea` component.
- It adds `cinematicSettings` and `userColliderRange` for `CameraModeArea` properties.


When `CT_CINAMATIC` is selected, the renderer uses the `cinematicSettings`. The main property for cinematic settings is the `cameraEntity,` which is the entity to which the camera will be attached. 

The `cameraEntity's Transform` (which falls back to `Transform.Identity`) is used to take position and rotation and zoom by the `scale.z` value.

The other properties enable the player input for the camera controller with well-known ranges; if they are not assigned, the camera controller is disabled.

Note: The player input for the avatar is kept, and the player could accidentally exit the area. To avoid that, see the changes in `AvatarModifierArea`.



### AvatarModifierArea
Work in progress

<details>
    <summary>Difference between experimental version and original one</summary>

```diff
interface PBAvatarModifierArea {
    /** the 3D size of the region */
    area: Vector3 | undefined;
    /** user IDs that can enter and remain unaffected */
    excludeIds: string[];
    /** list of modifiers to apply */
    modifiers: AvatarModifierType[];
+    movementSettings?: AvatarMovementSettings | undefined;
+    /** if true, the player will be considered inside the area when they are within 0.3m of the area. default true */
+    useColliderRange?: boolean | undefined;
}

+interface AvatarMovementSettings {
+    controlMode?: AvatarControlType | undefined;
+    /** if not explicitly set, the following properties default to user's preference settings */
+    runSpeed?: number | undefined;
+    /** how fast the player gets up to speed or comes to rest. higher = more responsive */
+    friction?: number | undefined;
+    /** how fast the player accelerates vertically when not on a solid surface, in m/s. should normally be negative */
+    gravity?: number | undefined;
+    /** how high the player can jump, in meters. should normally be positive. gravity must have the same sign for jumping to be possible */
+    jumpHeight?: number | undefined;
+    /** max fall speed in m/s. should normally be negative */
+    maxFallSpeed?: number | undefined;
+    /** speed the player turns in tank mode, in radians/s */
+    turnSpeed?: number | undefined;
+    /** speed the player walks at, in m/s */
+    walkSpeed?: number | undefined;
+    /** whether to allow player to move at a slower speed (e.g. with a walk-key or when using a gamepad/joystick). defaults to true */
+    allowWeightedMovement?: boolean | undefined;
+}
```
</details>


## Examples
Copy one of the following code to the index.ts and then run the Bevy Explorer preview with `npm start`.
<details>
  <summary>Cinematic camera example with lemminiscate curve example</summary>

  ```typescript
import { CameraModeArea, CameraType, engine, Entity, Material, MeshRenderer, PBCameraModeArea, Transform } from "@dcl/sdk/ecs";
import { Color3, Color4, Quaternion, Scalar, Vector3 } from "@dcl/sdk/math";

function colorAddAlpha(baseColor: Color4 | Color3, a: number): Color4 {
    return { ...baseColor, a }
}

function createCameraModeArea(position: Vector3, size: Vector3, value: Partial<PBCameraModeArea>, debug: boolean = true) {
    // When debug=true, it enables a box with alpha representation
    if (debug) {
        const height = 0.1
        const floorRepresentationEntity = engine.addEntity()
        const floorPosition = Vector3.create(position.x, position.y - size.y / 2 + height, position.z)
        MeshRenderer.setBox(floorRepresentationEntity)
        Material.setPbrMaterial(floorRepresentationEntity, { albedoColor: colorAddAlpha(Color4.Magenta(), 0.2) })
        Transform.create(floorRepresentationEntity, { position: floorPosition, scale: Vector3.create(size.x, height, size.z) })

        const areaRepresentationEntity = engine.addEntity()
        MeshRenderer.setBox(areaRepresentationEntity)
        Material.setPbrMaterial(areaRepresentationEntity, { albedoColor: colorAddAlpha(Color4.Green(), 0.1) })
        const repPosition = Vector3.create(position.x, position.y, position.z)
        Transform.create(areaRepresentationEntity, { position: repPosition, scale:  size})
    }

    const entity = engine.addEntity()
    CameraModeArea.create(entity, {
        mode: CameraType.CT_FIRST_PERSON,
        area: size,
        ...value,
    })
    Transform.create(entity, { position })
}

function createLemniscateMovement(centerPosition: Vector3, height: number, pathLength: number, periodSeg: number = 1.0, showDebug: boolean = false): Entity {
    const movingEntity = engine.addEntity()
    const systemName = `${movingEntity}-bernoulli-lemniscate-curve-movement-system`
    const speedModifier = 2 * Math.PI / periodSeg

    // A small debug box to show the position of the entity moved and the boundary of the path
    if (showDebug) {
        const debug = engine.addEntity()
        const debugBoxSize = 0.1
        MeshRenderer.setBox(debug)
        Material.setPbrMaterial(debug, { albedoColor: Color4.Red() })
        Transform.create(debug, { scale: Vector3.create(debugBoxSize, debugBoxSize, debugBoxSize), parent: movingEntity })

        const debugArea = engine.addEntity()
        MeshRenderer.setBox(debugArea)
        Material.setPbrMaterial(debugArea, { albedoColor: colorAddAlpha(Color4.Magenta(), 0.2) })
        Transform.create(debugArea, { scale: Vector3.create(pathLength, height, pathLength * Math.sqrt(2) / 4), position: {...centerPosition} })
    }
    
    let t = 0
    const amplitude = pathLength / 2
    engine.addSystem((dt) => {
        const transform = Transform.getMutableOrNull(movingEntity)
        // auto clean
        if (!transform) {
            engine.removeSystem(systemName)
            return
        }

        // t is acummulated time but periodic each 2pi
        t = Scalar.repeat(t + (speedModifier * dt), 2 * Math.PI)
        const ct = Math.cos(t)
        const st = Math.sin(t)
        const previousPos = {...transform.position}

        // lemminiscate curve
        transform.position.x = centerPosition.x + (amplitude * ct / (1 + (st * st)))
        transform.position.z = centerPosition.z + (amplitude * ct * st / (1 + (st * st)))

        // sin square 
        transform.position.y = centerPosition.y - height / 2 + (height * Math.sin(t/2) * Math.sin(t/2))

        // rotation calculated from the previous position (with forward vector)
        Quaternion.fromLookAtToRef(previousPos, transform.position, Vector3.Up(), transform.rotation)
    }, 0, systemName)

    // Initial position
    Transform.create(movingEntity, { position: {...centerPosition} })

    return movingEntity
}

export function main() {
    console.log("## Cinematic Example Test ##")

    // Camera area position and size
    const cameraAreaPosition = Vector3.create(4, 2.5, 4)
    const cameraAreaSize = Vector3.create(3, 3, 3)

    // Movement parameters
    const movementCenteredPosition = Vector3.create(8, 2, 8)
    const pathHeight = 1.0
    const pathLength = 4.0
    const periodSeg = 6

    const movingEntity = createLemniscateMovement(movementCenteredPosition, pathHeight, pathLength, periodSeg, true)
    createCameraModeArea(cameraAreaPosition, cameraAreaSize, {
        mode: CameraType.CT_CINEMATIC,
        cinematicSettings: {
            cameraEntity: movingEntity
        }
    })
}
  ```
</details>
