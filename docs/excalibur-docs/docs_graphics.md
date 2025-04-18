Skip to main content
On this page
Graphics are special object that abstract something that can be drawn to the screen. Examples include circles, polygons, and sprites.
warning
The GraphicsComponent centers graphics by default, set the actor.graphics.anchor to change this behavior.
## Graphics​
Excalibur `ex.Graphic`s break into 2 main categories, Sprite based and Raster based. That is to say Raster and non-Raster graphics.
All Graphics have a bunch of useful feature that work for ALL types of graphics
  * `clone(): Graphic`
    * Returns a new deep copy of this graphic
  * `width: number`
    * Gets or sets the width of the graphic
  * `height: number`
    * Gets or sets the height of the graphic
  * `flipHorizontal: boolean`
    * Gets or sets the flipHorizontal, which will flip the graphic horizontally (across the y axis)
  * `flipVertical: boolean`
    * Gets or sets the flipVertical, which will flip the graphic vertically (across the x axis)
  * `rotation: number`
    * Gets or sets the rotation of the graphic (in radians)
  * `opacity: number`
    * Gets or sets the opacity of the graphic, 0 is transparent, 1 is solid (opaque).
  * `scale: Vector`
    * Gets or sets the scale of the graphic, this effects the width and height of the graphics
  * `origin: Vector`
    * Gets or sets the origin of the graphic, if not set the center of the graphic is the origin
  * `tint: Color`
    * Applies a multiplicative tint to the pixels in the graphic. This is useful if you want to quickly tweak the color or a graphic without making new assets.


note
Multiplicative tint this means you cannot apply a `Color.White` tint to a graphic that will have an effect, under the hood the white color is represented as `(1, 1, 1)` and 1 multiplied against anything is itself.
### Raster Graphics​
Rasters are a type of `ex.Graphic` built by constructing a bitmap (using CanvasRenderingContext2D) in memory which is then sent to the drawing context. This allows the use of features available to developers in the 2D canvas to produce Graphics for Excalibur. It's important to share or limit the number of Raster graphics due to their memory footprint in the browser.
Rasters are only rendered when they need to be, if no properties change on them, they are not recalculated. Rasters can be forced to be re-draw by calling `.flagDirty()`.
See these useful raster implemenations
  * Text
  * Polygon
  * Rectangle
  * Circle
  * Canvas


A custom raster can be built implementing the `execute` function, this is in fact how the Canvas is implemented.
```

typescript
exportclassMyRasterextendsex.Raster {
constructor() {
super();
 }
execute(ctx:CanvasRenderingContext2D):void {
// my custom raster
  ctx.fillStyle ='blue';
  ctx.fillRect(0, 0, 20, 20);
 }
}
Copy
```
```

typescript
exportclassMyRasterextendsex.Raster {
constructor() {
super();
 }
execute(ctx:CanvasRenderingContext2D):void {
// my custom raster
  ctx.fillStyle ='blue';
  ctx.fillRect(0, 0, 20, 20);
 }
}
Copy
```

`ex.Raster` properties and methods
  * `abstract execute(ctx: CanvasRenderingContext2D): void`
    * This is the bitmap generation implementation, this will construct the desired bitmap on the CanvasRenderingContext2D.
  * `rasterize(): void`
    * This causes the underlying bitmap to be generated calling the `execute()` implementation in the process


### Polygon​
Polygon creates a rastered polygon graphic given a set of points
```

typescript
consttriangle=new ex.Polygon({
 points: [ex.vec(10*5, 0), ex.vec(0, 20*5), ex.vec(20*5, 20*5)],
 color: ex.Color.Yellow
});
Copy
```
```

typescript
consttriangle=new ex.Polygon({
 points: [ex.vec(10*5, 0), ex.vec(0, 20*5), ex.vec(20*5, 20*5)],
 color: ex.Color.Yellow
});
Copy
```

### Circle​
Circle creates a rastered circle graphic given a raidus
```

typescript
constcircle=new ex.Circle({
 radius: 10,
 color: ex.Color.Red
});
Copy
```
```

typescript
constcircle=new ex.Circle({
 radius: 10,
 color: ex.Color.Red
});
Copy
```

### Rectangle​
Rectangle creates a rastered rectange graphic given a width and height
```

typescript
constrect=new ex.Rectangle({
 width: 100,
 height: 100,
 color: ex.Color.Green
});
Copy
```
```

typescript
constrect=new ex.Rectangle({
 width: 100,
 height: 100,
 color: ex.Color.Green
});
Copy
```

### Canvas​
The canvas is a special type of raster graphic that acts like a shim between direct CanvasRenderingContext2D drawing and the ExcaliburGraphicsContext
This type of raster is re-rendered every time the graphic is drawn, thus it should be used sparingly due to this inefficiency.
```

typescript
constcanvas=newCanvas({
draw: (ctx:CanvasRenderingContext2D) => {
...
 }
});
Copy
```
```

typescript
constcanvas=newCanvas({
draw: (ctx:CanvasRenderingContext2D) => {
...
 }
});
Copy
```

  * Graphics
    * Raster Graphics
    * Polygon
    * Circle
    * Rectangle
    * Canvas


