Skip to main content
On this page
HTML is super great at building UI's for the web, there are many popular front end frameworks and tool kits that will deliver highly polished UIs, so we recommend that first.
Please consider html based first, but if the UI makes more sense in-canvas look to screen elements.
Advantages of HTML
  * Accessibility
  * Built in keyboard/pointer events
  * Powerful layout engine
  * CSS styling


## Using HTML with Excalibur's Canvas​
A common technique for building UI wit canvas based games is overlaying HTML elements over the canvas.
Check out this full demo excaliburjs/sample-html
```

html
<!DOCTYPEhtml>
<htmllang="en">
<head>
 <style>
html,body {
padding: 0;
margin: 0;
background-color: black;
  }
/* optionally center the game */
#game {
position: absolute;
left: 50%;
top: 50%;
transform: translate(-50%, -50%);
  }
.menu {
position: absolute;
top: 10px;
left: 10px;
  }
 </style>
</head>
<body>
 <main>
  <canvasid="game"></canvas>
  <divclass="menu">
   <button>Add Unit</button>
  </div>
 </main>
</body>
</html>
Copy
```
```

html
<!DOCTYPEhtml>
<htmllang="en">
<head>
 <style>
html,body {
padding: 0;
margin: 0;
background-color: black;
  }
/* optionally center the game */
#game {
position: absolute;
left: 50%;
top: 50%;
transform: translate(-50%, -50%);
  }
.menu {
position: absolute;
top: 10px;
left: 10px;
  }
 </style>
</head>
<body>
 <main>
  <canvasid="game"></canvas>
  <divclass="menu">
   <button>Add Unit</button>
  </div>
 </main>
</body>
</html>
Copy
```

When positioning elements over the Excalibur canvas, it is important to set `pointer-events: none` on elements you want to pass through to Excalibur. For elements you want to receive clicks/taps be sure to set `pointer-events: all`.
## Positioning HTML Elements with Excalibur​
Excalibur provides an API for converting game positions in world/screen space to page coordinates.
```

typescript
constengine=new ex.Engine({...});
// Excalibur camera is centered at (100, 100)
engine.currentScene.camera.pos = ex.vec(100, 100); 
// Finds the absolution page position that corresponds to the excalibur position (100, 100)
constpagePositionFromWorld= engine.screen.worldToPageCoordinates(ex.vec(100, 100)); 
// Finds the absolute page position that corresponds to the top left of the excalibur canvas, screen (0, 0)
constpagePositionFromScreen= engine.screen.screenToPageCoordinates(ex.vec(0, 0));
// Use pagePositions in page coordinates to position HTML elements
// Setting CSS variables is a convenient way to do this
document.documentElement.style.setProperty('--pointer-x', evt.pagePos.x.toString() +'px');
document.documentElement.style.setProperty('--pointer-y', evt.pagePos.y.toString() +'px');
Copy
```
```

typescript
constengine=new ex.Engine({...});
// Excalibur camera is centered at (100, 100)
engine.currentScene.camera.pos = ex.vec(100, 100); 
// Finds the absolution page position that corresponds to the excalibur position (100, 100)
constpagePositionFromWorld= engine.screen.worldToPageCoordinates(ex.vec(100, 100)); 
// Finds the absolute page position that corresponds to the top left of the excalibur canvas, screen (0, 0)
constpagePositionFromScreen= engine.screen.screenToPageCoordinates(ex.vec(0, 0));
// Use pagePositions in page coordinates to position HTML elements
// Setting CSS variables is a convenient way to do this
document.documentElement.style.setProperty('--pointer-x', evt.pagePos.x.toString() +'px');
document.documentElement.style.setProperty('--pointer-y', evt.pagePos.y.toString() +'px');
Copy
```

Then to position any elements reference the CSS variables in your CSS
```

css
.menu {
position: absolute;
/* position menu on click */
left: var(--pointer-x);
top: var(--pointer-y);
}
Copy
```
```

css
.menu {
position: absolute;
/* position menu on click */
left: var(--pointer-x);
top: var(--pointer-y);
}
Copy
```

## Scaling between CSS & Excalibur Pixels​
It is useful in CSS to scale your elements to match the Excalibur game canvas, this is done by calculating the ratio between between Excalibur pixels and browser pixels.
```

typescript
constcalculateExPixelConversion= (screen:ex.Screen) => {
constorigin= screen.worldToPageCoordinates(Vector.Zero);
constsinglePixel= screen.worldToPageCoordinates(vec(1, 0)).sub(origin);
constpixelConversion= singlePixel.x;
 document.documentElement.style.setProperty('--pixel-conversion', pixelConversion.toString());
}
// Update pixel conversion on resize
game.screen.events.on('resize', () =>calculateExPixelConversion(game.screen));
// Set initial conversion
game.start().then(() => {
calculateExPixelConversion(game.screen);
});
Copy
```
```

typescript
constcalculateExPixelConversion= (screen:ex.Screen) => {
constorigin= screen.worldToPageCoordinates(Vector.Zero);
constsinglePixel= screen.worldToPageCoordinates(vec(1, 0)).sub(origin);
constpixelConversion= singlePixel.x;
 document.documentElement.style.setProperty('--pixel-conversion', pixelConversion.toString());
}
// Update pixel conversion on resize
game.screen.events.on('resize', () =>calculateExPixelConversion(game.screen));
// Set initial conversion
game.start().then(() => {
calculateExPixelConversion(game.screen);
});
Copy
```

One of the lowest effort options is to apply a CSS transform to the container of your HTML UI based on the excalibur pixel conversion.
```

css
.excalibur-scale {
/* transform from the top left of the element */
transform-origin: 00;
/* scale the ui */
transform: scale(var(--pixel-conversion), var(--pixel-conversion));
}
Copy
```
```

css
.excalibur-scale {
/* transform from the top left of the element */
transform-origin: 00;
/* scale the ui */
transform: scale(var(--pixel-conversion), var(--pixel-conversion));
}
Copy
```

Another option is to manually apply conversion to specific CSS properties.
```

css
.text {
/* Convert to excalibur 24px from page 24px */
font-size: calc(24px*var(--pixel-conversion));
}
Copy
```
```

css
.text {
/* Convert to excalibur 24px from page 24px */
font-size: calc(24px*var(--pixel-conversion));
}
Copy
```

  * Using HTML with Excalibur's Canvas
  * Positioning HTML Elements with Excalibur
  * Scaling between CSS & Excalibur Pixels


