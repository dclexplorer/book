# Exposing internal GLTF nodes
This experimental feature enables you to access to the internal structure of a GLTF/GLB. Some of the use cases are:
- Use pointer events for specific colliders inside the GLTF
- Make code-animations (by modifying internal nodes transform)
- Modify internal materials
- Use internal meshes and put a video texture
- Not guessing the animation clip names (now you can fetch the animation list)
- Attach to internal entities

## What it changes
This proposal:
- Add properties into the `GltfLoadingState` to be able to fetch the internal structure
- Add the `GltfNode` and `GltfNodeState` component 
- Add the `gltf` property in the `Material` component
- Add the `GltfMesh` option to the `MeshRenderer` and `MeshCollider`

### GltfLoadingState

<details>
  <summary>Difference between experimental version and original one</summary>

```diff
export interface PBGltfContainerLoadingState {
+    animationNames: string[];
    currentState: LoadingState;
+    materialNames: string[];
+    meshNames: string[];
+    nodePaths: string[];
+    skinNames: string[]; // @deprecated this will not be finally used
}
```

</details>

Summary: 
- It adds the `animationNames` property, here you have the entire list of animation available from your GLTF
- It adds the `materialNames` property, you can fetch the material name to be referenced in the material's `gltf` property
- It adds the `meshNames` property, you can fetch the mesh name to be referenced in the MeshRenderer and MeshCollider `gltf` property
- It adds the `nodePaths` property, you can fetch the node name to be referenced in the GltfNode `path` property


### Material

<details>
  <summary>Difference between experimental version and original one</summary>

```diff
export interface PBMaterial {
+    gltf?: PBMaterial_GltfMaterial | undefined;
    material?: {
        $case: "unlit";
        unlit: PBMaterial_UnlitMaterial;
    } | {
        $case: "pbr";
        pbr: PBMaterial_PbrMaterial;
    } | undefined;
}
```

</details>

Summary: 
- It adds the `gltf` material reference, you can reference a material inside a GLTF exposed in `GltfLoadingState.materialNames`. When the gltfMaterial is set, the fallback values for undefined `pbr` or `unlit` are the GLTF ones instead of the default specified in the documentation. If you want to set the original default, you have to do it explicitly. 

### MeshRenderer

<details>
  <summary>Difference between experimental version and original one</summary>

```diff
+ MeshRenderer.setGltfMesh(...)

+  /**
+   * @public
+   * Set a gltf internal mesh in the MeshCollider component
+   * @param entity - entity to create or replace the MeshRenderer component
+   * @param source - the path to the gltf
+   * @param meshName - the name of the mesh in the gltf
+   */
+  setGltfMesh(entity: Entity, source: string, meshName: string, colliderLayers?: ColliderLayer | ColliderLayer[]): void
```

</details>

Summary: 
- Now you can set a MeshRenderer from a mesh resource inside a GltfContainer


### MeshCollider

<details>
  <summary>Difference between experimental version and original one</summary>

```diff
+ MeshCollider.setGltfMesh(...)

+  /**
+   * @public
+   * Set a gltf internal mesh in the MeshCollider component
+   * @param entity - entity to create or replace the MeshCollider component
+   * @param source - the path to the gltf
+   * @param meshName - the name of the mesh in the gltf
+   * @param colliderMask - the set of layer where the collider reacts, default: Physics and Pointer
+   */
+  setGltfMesh(entity: Entity, source: string, meshName: string, colliderLayers?: ColliderLayer | ColliderLayer[]): void
```

</details>

Summary: 
- Now you can set a MeshCollider from a mesh resource inside a GltfContainer

### GltfNode and GltfNodeState
<details>
  <summary>Experimental component added diff</summary>

```diff
+export interface PBGltfNode {
+    path: string;
+}
+export interface PBGltfNodeState {
+    error?: string | undefined;
+    state: GltfNodeStateValue;
+}
+export const enum GltfNodeStateValue {
+    GNSV_FAILED = 1,
+    GNSV_PENDING = 0,
+    GNSV_READY = 2
+}
```
</details>

These new two components are used to map a internal gltf node into an Entity. It MUST be a child from the `GltfContainer` and after you create the component, the renderer will create the `Transform`, `Material`, `MeshRenderer` and `MeshCollider` if apply. This enables you to receive the updates in case an animation is playing and that node is affected, or to modify them by yourself.


# Examples
You can check the examples scenes in the [examples repo](https://github.com/dclexplorer/experimental-example-scenes)

The folders starting with `gltf-example` are the ones using this experimental feature.

<video controls>
  <source src="../0804.mp4" type="video/mp4">
</video>