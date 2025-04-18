Skip to main content
On this page
Ray's are a useful tool for testing if geometry intersects a line of sight with Collider.rayCast. You can think of them as a point in space with a direction.
Examples where ray casting can be useful
  * Custom physics implementations (platformers, slopes, etc)
  * Line of sight AI
  * Proximity detection


warning
If you start a raycast from inside an Actor it will hit that Actor first unless you explicitly exclude it. See Raycast Options below.
## Scene Testing​
It is possible to test against all actors in a Scene at once.
```

typescript
// given a scene reference
constgame=new ex.Engine({...});
game.start();
constray=new ex.Ray(ex.vec(100, 100), ex.Vector.Right);
game.currentScene.physics.rayCast(ray, {...});
// or in a custom scene
classMySceneextendsex.Scene {
someRayTestMethod() {
constray=new ex.Ray(ex.vec(100, 100), ex.Vector.Right);
this.physics.rayCast(ray, {...});
 }
}
Copy
```
```

typescript
// given a scene reference
constgame=new ex.Engine({...});
game.start();
constray=new ex.Ray(ex.vec(100, 100), ex.Vector.Right);
game.currentScene.physics.rayCast(ray, {...});
// or in a custom scene
classMySceneextendsex.Scene {
someRayTestMethod() {
constray=new ex.Ray(ex.vec(100, 100), ex.Vector.Right);
this.physics.rayCast(ray, {...});
 }
}
Copy
```

## Scene Raycast Options​
Excalibur's raycast can be optionally configured in a variety of ways. By default if no options are provided to the `rayCast(ray, ...)` the options will be
```

typescript
consthits= scene.physics.rayCast(ray, {
 searchAllColliders: true,
 collisionGroup: CollisionGroup.All,
 collisionMask: CollisionGroup.All.mask,
 ignoreCollisionGroupAll: false,
 maxDistance: Infinity,
 filter: null,
})
Copy
```
```

typescript
consthits= scene.physics.rayCast(ray, {
 searchAllColliders: true,
 collisionGroup: CollisionGroup.All,
 collisionMask: CollisionGroup.All.mask,
 ignoreCollisionGroupAll: false,
 maxDistance: Infinity,
 filter: null,
})
Copy
```

### Search All Colliders​
By default the raycast will not stop after the first hit is found, if you'd like to only get the first hit set `searchAllColliders: false`
### Max Distance​
Distance is in terms of pixels, this is the maximum length along the ray to search for colliders.
### Hit Filtering​
Using the `filter` option you can do complex filtering logic to reject or accept potential hits.
```

typescript
scene.physics.rayCast(ray, {
filter: (potentialHit:RayCastHit) => {
// return true to accept the hit
// return false to reject the hit
returntrue;
 }
})
Copy
```
```

typescript
scene.physics.rayCast(ray, {
filter: (potentialHit:RayCastHit) => {
// return true to accept the hit
// return false to reject the hit
returntrue;
 }
})
Copy
```

### Collision Mask​
The collision mask it a bit mask with a 1 in the place for each collision group category you'd like to consider in the raycast. For example this is useful if all your enemies share a collision group category. See collision group documentation for more information.
You may want to pair this with `ignoreCollisionGroupAll: true` to avoid default actors which collide with everything including specific rayCast masks.
```

typescript
constplayerGroup=new ex.CollisionGroup('player', 0b0001, ~0b0001); 
constenemyGroup=new ex.CollisionGroup('enemy', 0b0010, ~0b0010);
...
// this raycast will only actors with the 1 in the second place, so enemyGroup
constenemyHits= scene.physics.rayCast(ray, {
 collisionMask: 0b0010;
});
// this raycast will only actors with the 1 in the first and second place, so playerGroup and enemyGroup
constplayerAndEnemyHits= scene.physics.rayCast(ray, {
 collisionMask: 0b0011;
});
Copy
```
```

typescript
constplayerGroup=new ex.CollisionGroup('player', 0b0001, ~0b0001); 
constenemyGroup=new ex.CollisionGroup('enemy', 0b0010, ~0b0010);
...
// this raycast will only actors with the 1 in the second place, so enemyGroup
constenemyHits= scene.physics.rayCast(ray, {
 collisionMask: 0b0010;
});
// this raycast will only actors with the 1 in the first and second place, so playerGroup and enemyGroup
constplayerAndEnemyHits= scene.physics.rayCast(ray, {
 collisionMask: 0b0011;
});
Copy
```

### Collision Group​
The `collisionGroup` searches for actors that match a specific collision group.
You may want to pair this with `ignoreCollisionGroupAll: true` to avoid default actors which collide with everything including specific rayCast groups.
## Specific Actor Geometry Testing​
If you have a specific actor or geometry that you know you will test against, all Collider types implement `rayCast(ray, [maxDistance])`.
```

typescript
constactor=new ex.Actor({
 pos: ex.vec(200, 100),
 width: 100,
 height: 100,
 color: ex.Color.Red,
})
constray=new ex.Ray(ex.vec(100, 100), ex.Vector.Right)
// Ray cast against the collider geometry, returns a point if intersects or null if not
constpoint= actor.collider.get().rayCast(ray)
// Vector(150, 100)
Copy
```
```

typescript
constactor=new ex.Actor({
 pos: ex.vec(200, 100),
 width: 100,
 height: 100,
 color: ex.Color.Red,
})
constray=new ex.Ray(ex.vec(100, 100), ex.Vector.Right)
// Ray cast against the collider geometry, returns a point if intersects or null if not
constpoint= actor.collider.get().rayCast(ray)
// Vector(150, 100)
Copy
```

  * Scene Testing
  * Scene Raycast Options
    * Search All Colliders
    * Max Distance
    * Hit Filtering
    * Collision Mask
    * Collision Group
  * Specific Actor Geometry Testing


