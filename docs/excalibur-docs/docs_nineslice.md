Skip to main content
On this page
warning
🧪 Available in the v0.30.0 Alpha
## Overview​
The NineSlice is a Graphic which uses a 2D technique allowing for reuse of an image at various sizes without needing to prepare multiple assets. The process includes taking an input sprite texture, and splitting it into 9 sections, which then can be repainted in a new sprite texture without scaling or distorting the texture. This is very useful in game development for UI panels or buttons or textured platforms.
For example: let's say this is your source texture sprite.
![Input Tile Image](https://excaliburjs.com/docs/nineslice)
The nine slice Graphic will break that texture up into 9 separate parts.
![sliced texture](https://excaliburjs.com/assets/images/9slice-a5a988fa3b622274bb896dba8126700d.png)
Then based on your output parameters specified it will 'redraw' this texture to whatever specified output graphic you want.
You can set parameters under `sourceConfig` properties to control the margins. This lets you dial in where the module 'slices' your texture up. For example, with the example 64 pixel x 64 pixel sprite, we can set the top/bottom margins to 5 pixels, and the left/right margins to 6 pixels to get this slicing.
![sliced](https://excaliburjs.com/assets/images/sliced-e9842760637cf3cdec853a09efe977e8.png)
## Configuration​
```

typescript
exporttypeNineSliceConfig=GraphicOptions& {
/*
 overwrites the GraphicOptions width/height param, as it is required for this module
 */
width:number;
height:number;
/*
 source is the loaded ImageSource, which you can load manually or through a loader 
 */
source:ImageSource;
/*
 sourceConfig has all the parameters necessary to cut up the input texture properly, width and height are the overall pixel dimensions of the texture and the margins define the overall cutting boundaries in pixels
 */
sourceConfig: {
width:number;
height:number;
topMargin:number;
leftMargin:number;
bottomMargin:number;
rightMargin:number;
 };
/*
 destination configuration specifies if you want center piece drawn, the '5' frame, and how you want the algorithm to manipulate the frame textures horizontally and vertically
 */
destinationConfig: {
drawCenter:boolean;
horizontalStretch:NineSliceStretch; //exported enum
verticalStretch:NineSliceStretch; //exported enum
 };
};
Copy
```
```

typescript
exporttypeNineSliceConfig=GraphicOptions& {
/*
 overwrites the GraphicOptions width/height param, as it is required for this module
 */
width:number;
height:number;
/*
 source is the loaded ImageSource, which you can load manually or through a loader 
 */
source:ImageSource;
/*
 sourceConfig has all the parameters necessary to cut up the input texture properly, width and height are the overall pixel dimensions of the texture and the margins define the overall cutting boundaries in pixels
 */
sourceConfig: {
width:number;
height:number;
topMargin:number;
leftMargin:number;
bottomMargin:number;
rightMargin:number;
 };
/*
 destination configuration specifies if you want center piece drawn, the '5' frame, and how you want the algorithm to manipulate the frame textures horizontally and vertically
 */
destinationConfig: {
drawCenter:boolean;
horizontalStretch:NineSliceStretch; //exported enum
verticalStretch:NineSliceStretch; //exported enum
 };
};
Copy
```

## Usage​
This is a Graphic object so it can be used on either an Actor or ScreenElement entity as a primary graphic, or as a part of a graphic group.
```

ts
exportclassPlayerextendsActor {
privatemyNineSlice:NineSlice;
constructor() {
super({ name: 'player', width: 300, height: 128, pos: newVector(100, 100) });
constmyNineSliceConfig:NineSliceConfig= {
   width: 300,
   height: 128,
   source: Resources.myImage,
   sourceConfig: {
    width: 64,
    height: 64,
    topMargin: 5,
    leftMargin: 6,
    bottomMargin: 5,
    rightMargin: 6
   },
   destinationConfig: {
    drawCenter: true,
    horizontalStretch: NineSliceStretch.TileFit,
    verticalStretch: NineSliceStretch.TileFit
   }
  };
this.myNineSlice =newNineSlice(myNineSliceConfig);
this.graphics.use(this.myNineSlice);
 }
}
Copy
```
```

ts
exportclassPlayerextendsActor {
privatemyNineSlice:NineSlice;
constructor() {
super({ name: 'player', width: 300, height: 128, pos: newVector(100, 100) });
constmyNineSliceConfig:NineSliceConfig= {
   width: 300,
   height: 128,
   source: Resources.myImage,
   sourceConfig: {
    width: 64,
    height: 64,
    topMargin: 5,
    leftMargin: 6,
    bottomMargin: 5,
    rightMargin: 6
   },
   destinationConfig: {
    drawCenter: true,
    horizontalStretch: NineSliceStretch.TileFit,
    verticalStretch: NineSliceStretch.TileFit
   }
  };
this.myNineSlice =newNineSlice(myNineSliceConfig);
this.graphics.use(this.myNineSlice);
 }
}
Copy
```

## Stretch Options​
There are two parameters in the configuration that need to be set to dictate the behavior of the drawing destination graphic, `horizontalStretch` and `verticalStretch`. They are of type `NineSliceStretch`. The `horizontalStretch` parameter will dictate the behavior of tile slices 2,5, and 8, as these frames can stretch horizontally. The `verticalStretch` parameter dictates how tile slices 4,5,6 behave, as they can stretch vertically.
The class includes some exported enumerated options:
```

ts
exportenumNineSliceStretch {
 Stretch,
 Tile,
 TileFit
}
Copy
```
```

ts
exportenumNineSliceStretch {
 Stretch,
 Tile,
 TileFit
}
Copy
```

### Stretch​
![Stretched Texture](https://excaliburjs.com/docs/nineslice)
The stretch setting takes input slice and paints in across the entirety of the output dimensions, causing a stretching of the texture as its redrawn, but this is guaranteed to fit the designated area, just with some potential image distortion.
### Tile​
![Tiled Texture](https://excaliburjs.com/docs/nineslice)
The tile setting takes input slice and paints in across the entirety of the output dimensions in a repeating texture fashion that will not distort the source image at all, but may not fit properly in the destination space, leaving a potentially partial drawing of the texture remaining.
### TileFit​
![Tile Fitted Texture](https://excaliburjs.com/docs/nineslice)
The tile fit setting takes the input slice and paints it across the entirety of the output dimensions in a repeating texture, but it 'resizes' the output tiling to create a 'best fit' of the texture, which is calculated to fit properly, but may have some squished and distorted texture.
## Draw Center​
```

ts
destinationConfig: {
drawCenter: boolean; //<------------ this property
...
 };
Copy
```
```

ts
destinationConfig: {
drawCenter: boolean; //<------------ this property
...
 };
Copy
```

The boolean flag `drawCenter` under `destinationConfig` tells the module if the section 5, middle section should be drawn or not, if this is set to `false` then it gets skipped in its draw routine, and remains 'transparent' so that the NineSlice is more of a border or frame Graphic.
![No Center](https://excaliburjs.com/docs/nineslice)
Look, Ma! No Center!
## Changing the Graphic​
At this point, if you modify the graphic details, it is recommended to recreate a new instance of the graphic and update your Actor or Screen Element if it needs changed. The existing instance will not dynamically update itself.
  * Overview
  * Configuration
  * Usage
  * Stretch Options
    * Stretch
    * Tile
    * TileFit
  * Draw Center
  * Changing the Graphic


