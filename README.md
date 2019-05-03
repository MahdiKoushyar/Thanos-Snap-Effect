# Thanos-Snap-Effect
Thanos Snap Effect Using html2canvas.js and chance.js
# First Step – Convert Element to Image

```
<div class="content">
     <img src="yourimagehere.png" height="600">
     <button id="start-btn">Snap!</button>
</div>

```
Fortunately, we have a very useful library calls html2canvas. You can just pass any html element and it will return canvas object for you. Once we have included the library, we’ll pass our div element to get the canvas object. Then we’ll get an array containing all pixels data from it.
```
html2canvas($(".content")[0]).then(canvas => {
    //capture all div data as image
    ctx = canvas.getContext("2d");
    var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    var pixelArr = imageData.data;

```
# Second Step – Chop Them to Pieces
We have the pixels array, we’ll try to distribute the pixels data to multiple canvases. But this is a little bit tricky. Since we want the animation to start fading away from top to bottom, we need majority pixels at the top of burger to be in the first group of canvases. And majority of bottom pixels in the last canvases group. This way, when we start each canvas’ animation sequentially , it will look like it’s fading from top to bottom.

The problem is this is not a regular random anymore so we can’t just use Math.random . What I did was creating a weighted distribution function. Basically we’ll just increase the probability for the top pixels to be in the first canvases group and the bottom to be in the last.

To achieve this, I also use chance.js A JavaScript library dedicated for random utility.

```
function weightedRandomDistrib(peak) {
  var prob = [], seq = [];
  for(let i=0;i<canvasCount;i++) {
    prob.push(Math.pow(canvasCount-Math.abs(peak-i),3));
    seq.push(i);
  }
  return chance.weighted(seq, prob);
}

```
Now we have the distributed pixels data. we’ll create canvases from them,and assign a class name. Then append them to the wrapper.

```
//put pixel info to imageDataArray (Weighted Distributed)
for (let i = 0; i < pixelArr.length; i+=4) {
  //find the highest probability canvas the pixel should be in
  let p = Math.floor((i/pixelArr.length) *canvasCount);
  let a = imageDataArray[weightedRandomDistrib(p)];
  a[i] = pixelArr[i];
  a[i+1] = pixelArr[i+1];
  a[i+2] = pixelArr[i+2];
  a[i+3] = pixelArr[i+3]; 
}
//create canvas for each imageData and append to target element
for (let i = 0; i < canvasCount; i++) {
  let c = newCanvasFromImageData(imageDataArray[i], canvas.width, canvas.height);
  c.classList.add("dust");
  $(".wrapper").append(c);
}

```
# Last Step – The Animation

The last step is to add the animation. first we’ll start fading away the original content using jQuery fadeout.

```
//clear all children except the canvas
$(".content").children().not(".dust").fadeOut(3500);
```

Then for each canvas, we’ll add three animations. First is blur. We add this to soften the transform or it will look pixelated. The second is transform. This is to move the pixels away from the original position. We add both rotation and translate using random value to simulate dust effect. And the third is fadeout to fade away the dust particle.

```
//apply animation
$(".dust").each( function(index){
  animateBlur($(this),0.8,800);
  setTimeout(() => {
    animateTransform($(this),100,-100,chance.integer({ min: -15, max: 15 }),800+(110*index));
  }, 70*index); 
  //remove the canvas from DOM tree when faded
  $(this).delay(70*index).fadeOut((110*index)+800,"easeInQuint",()=> {$( this ).remove();});
});

```
The tricky part is jQuery doesn’t directly support blur or transform animation so I have to manually create a function for them (See full source code in repository)

On the CSS side There is nothing much. Just flex display to center everything and basic style for the snap button. The only thing related to the effect is the position absolute. This is to make all the canvases stay on the same position. The rest of the animation are handled by the JavaScript.

```
.dust {
  position: absolute;
}
```


