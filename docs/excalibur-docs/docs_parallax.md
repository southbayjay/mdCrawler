Skip to main content
The ParallaxComponent can be used for creating parallax effects on the graphics, entities with this component are drawn differently and a collider will not be where you expect. It is not recommended you use colliders with parallax entities.
```

typescript
constactor=new ex.Actor()
// The actor will be drawn shifted based on the camera position scaled by the parallax factor
actor.addComponent(newParallaxComponent(ex.vec(0.5, 0.5)))
Copy
```
```

typescript
constactor=new ex.Actor()
// The actor will be drawn shifted based on the camera position scaled by the parallax factor
actor.addComponent(newParallaxComponent(ex.vec(0.5, 0.5)))
Copy
```

