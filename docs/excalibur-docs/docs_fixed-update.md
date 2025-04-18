Skip to main content
On this page
Using a fixed update time step can be useful when you need a very stable and consistent physics simulation, for example if you have a precision platformer or a physics game (think billiards or angry birds).
```

typescript
var game =new ex.Engine({
 fixedUpdateFps: 30,
})
Copy
```
```

typescript
var game =new ex.Engine({
 fixedUpdateFps: 30,
})
Copy
```

Drawn graphics using the GraphicsComponent will automatically be interpolated to avoid any jitter between frames if the fixed update FPS is less than the display FPS.
This interpolation can be disabled on a per actor basis
```

typescript
// turn off interpolation on a per actor basis
constactor=new ex.Actor({...});
actor.body.enableFixedUpdateInterpolate =false;
Copy
```
```

typescript
// turn off interpolation on a per actor basis
constactor=new ex.Actor({...});
actor.body.enableFixedUpdateInterpolate =false;
Copy
```

## Implications​
When fixed update is enabled the update will run at a guaranteed time step to meet the configured FPS in `fixedUpdateFps`.
  1. If the display FPS is greater than the configured fixed update FPS


  * The update may be skipped during the frame in order to satisfy the configured fixed update.
  * For example, if the display is running at 60FPS and the fixed update is 30FPS, then the update will only run every other frame.


  1. If the configured fixed update FPS is greater than the display FPS


  * The update may be run multiple times in a frame in order to satisfy the configured fixed update.
  * For example, if the display is running at 30FPS and the fixed update is 60FPS, then the update will run twice every frame to catch up.


## Performance​
If you can get away with a lower fixed FPS update you can save on some performance at the cost of simulation quality.
On other side, you may desire a higher quality physics simulation for your game, but it will come at the cost of performance.
  * Implications
  * Performance


