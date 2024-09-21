# Lights

This feature introduces enhanced control over lighting in your scenes, allowing developers to modify global light settings, as well as create point lights and spotlights attached to non-root entities. Some key use cases include:
- Override the default global directional light (sunlight) for dynamic lighting environments
- Create point lights (e.g., lightbulbs) or spotlights for localized light sources
- Control shadows, brightness, and color for more realistic scene rendering

Also this new feature is integrated with the `Exposing internal GLTF nodes` allowing modyfing Lights from GLTF or add to entities there as well.

## What it changes

This proposal:
- Adds the `PBLight` component for defining both global and local light sources
- Adds the `PBSpotlight` component for creating spotlights
- Adds the `PBGlobalLight` component for controlling scene-wide ambient and directional light settings (only for Root entity)

### PBLight

<details>
  <summary>New declaration</summary>

```diff
+export interface PBLight {
+    color?: PBColor3 | undefined;
+    enabled?: boolean | undefined;
+    illuminance?: number | undefined;
+    shadows?: boolean | undefined;
+}
```

</details>

Summary:
- The `PBLight` component introduces a flexible interface for defining light sources. Lights can be enabled or disabled, adjusted for brightness (illuminance), and set to cast or ignore shadows.
- Lights can be attached to non-root entities to simulate localized light effects like lamps or bulbs.
- The `color` property allows you to specify the light color using the `PBColor3` format.
  
#### Illuminance
The `illuminance` property defines the light’s brightness in lux (lumens/m²). For global directional lights, this is applied uniformly. For point or spotlights, this specifies the illuminance at a 1m distance from the light source.

#### Shadows
The `shadows` property toggles whether a light casts shadows. The engine may limit the number of lights casting shadows based on performance settings.

### PBSpotlight

<details>
  <summary>New Declaration</summary>

```diff
+export interface PBSpotlight {
+    angle: number;
+    innerAngle?: number | undefined;
+}
```

</details>

Summary:
- The `PBSpotlight` component turns a point light into a spotlight, emitting light in a cone defined by the `angle` property.
- Use the `innerAngle` property to define a smooth fall-off for light intensity within the spotlight cone.
- Spotlights are useful for simulating directional light sources like torches or headlights.

#### Angle
The `angle` property (in radians) defines the width of the cone in which the light is emitted. For example, a typical torch would have an angle of around `0.15`.

#### Inner Angle
The `innerAngle` defines the core area of maximum brightness. The light intensity decreases smoothly between the `innerAngle` and `angle`.

### PBGlobalLight

<details>
  <summary>New Declaration</summary>

```diff
+export interface PBGlobalLight {
+    ambientBrightness?: number | undefined;
+    ambientColor?: PBColor3 | undefined;
+    direction?: PBVector3 | undefined;
+}
```

</details>

Summary:
- The `PBGlobalLight` component allows you to adjust the global ambient and directional lighting for the entire scene.
- You can modify the ambient light's brightness (`ambientBrightness`) and color (`ambientColor`), which are applied uniformly across the scene.
- The `direction` property changes the angle at which the global directional light (e.g., sunlight) is cast.

#### Direction
Use the `direction` property to define the directional light’s vector in the scene. This is especially useful for controlling the sunlight's position based on the time of day or desired atmosphere.

#### Ambient Light
The `ambientColor` and `ambientBrightness` properties control the global ambient lighting. Ambient light affects the entire scene and is not tied to any particular light source.

# Examples
You can find example scenes demonstrating the use of the new lighting components in the [examples repo](https://github.com/dclexplorer/experimental-example-scenes).

Look for folders prefixed with `lights` for examples using this feature.