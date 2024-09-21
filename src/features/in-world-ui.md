# In World UI

This feature introduces the `UiCanvasTexture` option and `PBUiCanvas` component, enabling developers to create and render UI elements onto a texture that can be used within the 3D world. 

## What it changes
This proposal:
- Adds the `UiCanvasTexture` option for `Texture`, allowing you to link a UI entity to a texture and configure properties such as wrap mode and filter mode.
- Adds the `UiCanvas` component, allowing the definition of the dimensions and background color of the UI canvas and to use as the root entity for an defined UI.
- Updates the `TextureUnion` to include the new `UiCanvasTexture` option.
- Introduces new methods in the `ReactEcsRenderer` for setting up UI textures.

### PBUiCanvas

<details>
  <summary>New typescript declaration</summary>

```diff
+ export interface PBUiCanvas {
+    color?: PBColor4 | undefined;
+    height: number;
+    width: number;
+ }
```
</details>

Summary:
- It adds the `PBUiCanvas` message, which allows you to define the width, height, and optional background color of the canvas. The color defaults to transparent (`0, 0, 0, 0`) if not provided.
- The canvas can be attached to a root entity, and the properties such as width, height, and color can be adjusted according to your needs.

### TextureUnion

<details>
  <summary>Difference between experimental version and original one</summary>

```diff
export interface TextureUnion {
    // (undocumented)
    tex?: {
        $case: "texture";
        texture: Texture;
    } | {
        $case: "avatarTexture";
        avatarTexture: AvatarTexture;
    } | {
        $case: "videoTexture";
        videoTexture: VideoTexture;
+    } | {
+        $case: "uiTexture";
+        uiTexture: UiCanvasTexture;
    } | undefined;
}
```
</details>

Summary:
- The `TextureUnion` interface is updated to include `uiTexture`, enabling the attachment of a UI canvas to a texture.

### ReactEcsRenderer

<details>
  <summary>New methods added to ReactEcsRenderer</summary>

```diff
+ interface ReactEcsRenderer {
+     setTextureRenderer: (entity, ui) => void
+ }
```
</details>

Summary:
- The `ReactEcsRenderer` interface now provides the `setTextureRenderer` method, allowing developers to set a texture renderer for a specific entity that is linked to a UI canvas.

## Example Usage
The following is an example of how to use the `UiCanvasTexture` and `PBUiCanvas` in practice.

```typescript
// true for test in-world-ui and false to get the regular ui
const useTexture = true

// Initialize the UI
const ui = new UiExample()

// Create an entity to serve as the UI canvas
const uiCanvas = engine.addEntity();
if (useTexture) {
  // Create a UI canvas with specified width, height, and background color
  UiCanvas.create(uiCanvas, { width: 1024, height: 1024, color: Color4.Black() });
  
  // Set the texture renderer to the created canvas
  ReactEcsRenderer.setTextureRenderer(uiCanvas, ui.render.bind(ui));
} else {
  // Set the default UI renderer
  ReactEcsRenderer.setUiRenderer(ui.render.bind(ui));
}
```

This example demonstrates how to create and render a UI canvas texture in a virtual world. You can either use the in-world UI as a texture applied to a 3D object or render a regular 2D UI.
