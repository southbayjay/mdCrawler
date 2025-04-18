Skip to main content
On this page
Cameras are attached to scenes and can be changed by setting Scene.camera. By default, a Scene is initialized with a Camera that doesn't move and is centered on the screen.
### Focus​
Cameras have a position (x, y) which means they center around a specific point.
If a camera is following an Actor, it will ensure the actor is always at the center of the screen. You can use Camera.x and Camera.y instead if you wish to offset the focal point.
### Camera zooming​
To adjust the zoom for your game, use Camera.zoom which will scale the game accordingly. You can pass a duration to transition between zoom levels.
This can be useful as a way to scale up your game.
### Camera strategies​
Cameras can implement a number of strategies to track, follow, or exhibit custom behavior in relation to a target. A common reason to use a strategy is to have the Camera follow an Actor.
In order to user the different built-in strategies, you can access Camera.strategy
warning
It is important to add the camera strategy in your Engine, Scene, or Actor's `onInitialize()`, the strategy will automatically update itself so do not put it in a update/draw lifecycle event.
Lock the camera exactly to the center of the actor's bounding box
```

typescript
game.currentScene.camera.strategy.lockToActor(actor);
Copy
```
```

typescript
game.currentScene.camera.strategy.lockToActor(actor);
Copy
```

Lock the camera to one axis of the actor, in this case follow the actors x position
```

typescript
game.currentScene.camera.strategy.lockToActorAxis(actor, ex.Axis.X);
Copy
```
```

typescript
game.currentScene.camera.strategy.lockToActorAxis(actor, ex.Axis.X);
Copy
```

Elastically move the camera to an actor in a smooth motion see ElasticToActorStrategy for details
```

typescript
game.currentScene.camera.strategy.elasticToActor(actor, cameraElasticity, cameraFriction);
Copy
```
```

typescript
game.currentScene.camera.strategy.elasticToActor(actor, cameraElasticity, cameraFriction);
Copy
```

Keep the actor within a circle around the focus
```

typescript
game.currentScene.camera.strategy.radiusAroundActor(actor, radius);
Copy
```
```

typescript
game.currentScene.camera.strategy.radiusAroundActor(actor, radius);
Copy
```

Keep the camera limited within camera constraints. Make sure that the camera bounds are at least as large as the viewport size.
```

typescript
let boundingBox =newBoundingBox(leftBorder, topBorder, rightBorder, bottomBorder);
game.currentScene.camera.strategy.limitCameraBounds(boundingBox);
Copy
```
```

typescript
let boundingBox =newBoundingBox(leftBorder, topBorder, rightBorder, bottomBorder);
game.currentScene.camera.strategy.limitCameraBounds(boundingBox);
Copy
```

#### Custom strategies​
Custom strategies can be implemented by extending the CameraStrategy interface and added to cameras to build novel behavior with `ex.Camera.addStrategy<T>(new MyCameraStrategy<T>())`.
As shown below a camera strategy calculates a new camera position vector every frame given a target type, camera, engine, and elapsed delta in milliseconds.
```

typescript
/**
 * Interface that describes a custom camera strategy for tracking targets
 */
exportinterfaceCameraStrategy<T> {
/**
  * Target of the camera strategy that will be passed to the action
  */
target:T;
/**
  * Camera strategies perform an action to calculate a new focus returned out of the strategy
  */
action: (target:T, camera:Camera, engine:Engine, delta:number) =>Vector;
}
Copy
```
```

typescript
/**
 * Interface that describes a custom camera strategy for tracking targets
 */
exportinterfaceCameraStrategy<T> {
/**
  * Target of the camera strategy that will be passed to the action
  */
target:T;
/**
  * Camera strategies perform an action to calculate a new focus returned out of the strategy
  */
action: (target:T, camera:Camera, engine:Engine, delta:number) =>Vector;
}
Copy
```

### Camera shake​
To add some fun effects to your game, the Camera.shake method will do a random shake. This is great for explosions, damage, and other in-game effects.
### Camera lerp​
"Lerp" is short for Linear Interpolation and it enables the camera focus to move smoothly between two points using timing functions. Use Camera.move to ease to a specific point using a provided EasingFunction.
```

typescript
exportinterfaceEasingFunction {
 (currentTime:number, startValue:number, endValue:number, duration:number):number;
}
// or any builtin
constlinear= ex.EasingFunctions.Linear;
consteasinQuad= ex.EasingFunctions.EaseInQuad;
// ... and more!
Copy
```
```

typescript
exportinterfaceEasingFunction {
 (currentTime:number, startValue:number, endValue:number, duration:number):number;
}
// or any builtin
constlinear= ex.EasingFunctions.Linear;
consteasinQuad= ex.EasingFunctions.EaseInQuad;
// ... and more!
Copy
```

  * Focus
  * Camera zooming
  * Camera strategies
  * Camera shake
  * Camera lerp


