# Scrollable UI
This experimental feature enables you to add scrollbars to UI in case of overflow the container size.

## What it changes
This proposal changed the `UiTransform` and add a new component `UiScrollResult`.

### UiTransform

<details>
  <summary>Difference between experimental version and original one</summary>

```diff
+/**
+ * @public
+ * The scroll-visible determines if the scrollbars are shown when the scroll overflow is enabled
+ */
+export type ScrollVisibleType = 'horizontal' | 'vertical' | 'both' | 'hidden'

export interface UiTransformProps {
    ...,
+  /** A reference value to identify the element, default empty */
+  elementId?: string
+  /** default position=(0,0) if it aplies, a vector or a reference-id */
+  scrollPosition?: Vector2 | string
+  /** default ShowScrollBar.SSB_BOTH */
+  scrollVisible?: ScrollVisibleType
}

```

</details>

Summary: 
- It adds `elementId` to identify each UI element, it's a unique identifier. You can use it to identify outside the UI system, or to reference it in the `scrollPosition`
- It adds `scrollPosition` to manually set the current scroll position. It can be a normalized 2D vector (0-1, 0-1) or a string to reference another UI element.
- It adds `scrollVisible` to set the bars visibility


### UiScrollResult
```diff
+export interface PBUiScrollResult {
+    value: PBVector2 | undefined;
+}
```

The value indicates how scrolled is the container where `0.0` is at the beginning and `1.0` full scrolled. The x-value is for horizontal axes and y-value for the vertical one.


## Examples
Copy one of the following code to the index.ts and then run the Bevy Explorer preview with `npm start`.
<details>

  <summary>  Scrollable UI with a Scroll Container  

  ![showing how the bevy explorer is downloading](../2-2-example-scrollable-ui.gif)
  
  </summary>



  ```typescript
import ReactEcs, { Label, UiEntity } from '@dcl/react-ecs'
import { UiScrollResult, UiTransform, engine } from '@dcl/sdk/ecs'
import { Color4 } from '@dcl/sdk/math'
import { Button, Input, ReactEcsRenderer } from '@dcl/sdk/react-ecs'
import { type Vector2 } from '~system/EngineApi'

class UiExample {
	// autoincrement counter, only for demonstration purposes
	private counter: number = 0
	// target for scroll position
	private target: string | Vector2 = { x: 0.5, y: 0.5 }
	// text to display in the scroll controller
	private scrollText = 'indeterminated'
	// id of the scroll container, to identify it in the controller
	private readonly scrollContainerId = 'my-scroll-container-A'

	constructor() {
		engine.addSystem(this.controllerSystem.bind(this))
	}

	controllerSystem(): void {
		for (const [, pos, uiTransform] of engine.getEntitiesWith(
			UiScrollResult,
			UiTransform
		)) {
			if (uiTransform.elementId !== this.scrollContainerId) {
				continue
			}

			if (pos.value === undefined) {
				break
			}

			if (pos.value.y <= 0) {
				this.scrollText = 'top'
			} else if (pos.value.y >= 1) {
				this.scrollText = 'bottom'
			} else if (pos.value.y < 0.5) {
				this.scrollText = 'near top'
			} else {
				this.scrollText = 'near bottom'
			}

			if (pos.value.x <= 0) {
				this.scrollText += ' left'
			} else if (pos.value.x >= 1) {
				this.scrollText += ' right'
			} else {
				this.scrollText += ' middle'
			}
		}

		this.counter++
	}

	// This UI is only for demonstration purposes, not the focus of this example
	ScrollController(): ReactEcs.JSX.Element {
		return (
			<UiEntity
				uiTransform={{
					flexDirection: 'column',
					position: { left: '25%', top: '10%' },
					width: '200',
					height: '300',
					justifyContent: 'space-evenly',
					alignItems: 'center'
				}}
				uiBackground={{ color: Color4.create(0.0, 0.0, 0.0, 1.0) }}
			>
				<Label value="Scroll controller" color={Color4.Green()} fontSize={14} />
				<Button
					fontSize={16}
					uiTransform={{ width: '80%' }}
					value="Focus first item"
					onMouseDown={() => {
						this.target = 'first'
					}}
				/>
				<Button
					fontSize={16}
					uiTransform={{ width: '80%' }}
					value="Focus second item"
					onMouseDown={() => {
						this.target = 'second'
					}}
				/>
				<Label
					fontSize={16}
					value={`Currently:\n${this.scrollText}`}
					color={Color4.White()}
				/>

				<Input
					fontSize={16}
					uiTransform={{ width: '90%' }}
					placeholder="type target"
					onChange={(value) => {
						console.log(`change ${value}`)
						this.target = value
					}}
					onSubmit={(value) => {
						console.log(`submit ${value}`)
						this.target = value
					}}
				/>
			</UiEntity>
		)
	}

	Scrolly(): ReactEcs.JSX.Element {
		return (
			<UiEntity
				uiTransform={{
					flexDirection: 'column',
					alignItems: 'center',
					justifyContent: 'space-between',
					positionType: 'absolute',
					width: '400',
					height: '600',
					position: { right: '8%', bottom: '3%' },

					// new properties
					overflow: 'scroll', // enable scrolling
					scrollPosition: this.target, // if you want to set the scroll position programatically (maybe an action from the user)
					elementId: this.scrollContainerId // id to identify the scroll result if you need to
				}}
				uiBackground={{
					color: Color4.White()
				}}
			>
				<Label
					uiTransform={{
						height: 'auto',
						width: 'auto',
						margin: '200px',
						padding: `10px`,
						// new property: we set the id, it must be unique, and we will use it to identify the scroll position
						elementId: 'first'
					}}
					value={`first (${this.counter})`}
					color={Color4.Black()}
					fontSize={18}
					textAlign="middle-center"
					key="first"
				/>
				<Label
					uiTransform={{
						height: 'auto',
						width: 'auto',
						margin: '200px',
						padding: `10px`,
						// new property: we set the id, it must be unique, and we will use it to identify the scroll position
						elementId: 'second'
					}}
					value="second"
					color={Color4.Black()}
					fontSize={18}
					textAlign="middle-center"
				/>
				<Label
					uiTransform={{
						height: 'auto',
						width: 'auto',
						margin: '200px',
						padding: `10px`,
						// new property: we set the id, it must be unique, and we will use it to identify the scroll position
						elementId: 'third'
					}}
					value="third"
					color={Color4.Black()}
					fontSize={18}
					textAlign="middle-center"
				/>
				<Label
					uiTransform={{
						height: 'auto',
						width: 'auto',
						margin: '200px',
						padding: `10px`,
						// new property: we set the id, it must be unique, and we will use it to identify the scroll position
						elementId: 'fourth'
					}}
					value="fourth"
					color={Color4.Black()}
					fontSize={18}
					textAlign="middle-center"
				/>
				<Label
					uiTransform={{
						height: 'auto',
						width: 'auto',
						margin: '200px',
						padding: `10px`,
						// new property: we set the id, it must be unique, and we will use it to identify the scroll position
						elementId: 'fifth'
					}}
					value="fifth"
					color={Color4.Black()}
					fontSize={18}
					textAlign="middle-center"
				/>
			</UiEntity>
		)
	}

	render(): ReactEcs.JSX.Element[] {
		return [this.Scrolly(), this.ScrollController()]
	}
}

export function main(): void {
	const ui = new UiExample()
	ReactEcsRenderer.setUiRenderer(ui.render.bind(ui))
}

  ```
</details>
