# PPP-U2NetTryout
Tryout for my personal passion project with U2Net via RunwayML

# Set up
## RunwayML

To use <a href="https://runwayml.com/">RunwayML</a> you need to make an account and download the desktop application (you can also start the application in the browser). With a "free plan", you get $10 credits you can use for hosting the model. That's a great start to run my try-out by yourself and test other models if you want to. 

- $0.05/minute for running models
- $0.005/step for training models (Keep in mind that training models require an upgrade to the "creative plan")

You can find more information about RunwayML and why to use it <a href="https://dev.to/jochemstoel/runwayml-next-generation-machine-learning-for-creators-1b4p">here</a>. 

## Simple HTML
Of course, we start off with a simple HTML file you can use to integrate the U2-Net model. We need 3 "main" items on the HTML page.
- A <canvas> with <video> where your video will be shown
- A <img> which displays the taken image
- A <img> which displays the returned U2-Net image without background

We also need 3 buttons:
- btnPicture: A button which takes the photo
- btnOk: You click this button if you want to choose that image to send to the U2-Net model
- btnRemove: This button will trigger the code to access and execute the U2-Net model

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>U2-Net tryout</title>
</head>
<style>
    .section{
        margin-top: 2rem;
        margin-bottom: 1rem;
    }
    .frames{
        display: flex;
    }
    .input-frame__canvas, .output-frame__canvas {
      width: 320px;
      height: 240px;
    }
    .hide{
        display: none;
    }
</style>
<body>
    <section class="section">
        <div class="frames">
          <div class="input-frame takePicture">
            <h3>TAKE A PICTURE</h3>
            <canvas id="iCanvas" class="input-frame__canvas" width="640" height="480"></canvas>
            <br/>
            <button id="btnPicture">TAKE PICTURE</button>
          </div>
          <div class="input-frame">
            <h3>PREVIEW</h3>
            <img id="iPic" width="320" height="240"></img>
            <br/>
            <button class="hide" id="btnOk">USE THIS PICTURE</button>
          </div>
        </div>
    </section>
    <section class="section removeBackground">
        <button id="btnRemove">REMOVE BACKGROUND</button>
        <br/>
        <img id="oImg" width="320" height="240"></img>
    </section>
</body>
</html>
```

## Adding the Javascript
The next step is to add javascript to execute the code. First, we will insert the javascript code to add a video tag to iCanvas.

``` javascript
<script>
  {
    const $iCanvas = document.getElementById('iCanvas');
    const iCtx = $iCanvas.getContext('2d');
    const $iCanvasSection = document.querySelector('.takePicture');

    const $iPic = document.getElementById('iPic');

    const $btnPicture = document.getElementById('btnPicture');
    const $btnOk = document.getElementById('btnOk');

    const $oImg = document.getElementById('oImg');
    const $btnRemove = document.getElementById('btnRemove');
    const $oImgSection = document.querySelector('.removeBackground');
    //hide the section until the button btnRemove is clicked
    $oImgSection.classList.add('hide');

    let $video, iCanvasData;

    const init = async() => {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
      $video = document.createElement('video');
      $video.muted = true;
      $video.autoplay = true;
      $video.playsinline = true;
      $video.srcObject = stream;

      if (!$video.captureStream) {
        $video.captureStream = () => stream;
      }

      $video.play();

      requestAnimationFrame(draw);
    }

    const draw = () => {
      iCtx.drawImage($video, 0,0);
      requestAnimationFrame(draw);
    };

    init();
  }
</script>
```

Next, we need to add a click function to btnPicture, which will take a snapshot of your video element and paste it into <img> iPic. In iCanvasData we save a base64 image, because that's requested as input by the model. You can see this in RunWayML after you added the model to the workspace. Then you click on "network > javascript" and see the provided javascript code to access the model.

``` javascript
<script>
  ...
  
  const init = async() => {
    ...
    video.play();
    
    $btnPicture.addEventListener('click', process);
    
    requestAnimationFrame(draw);
  }
  
  ...
  
  const process = () => {
    //Save the data from the canvas to BASE64
    iCanvasData = $iCanvas.toDataURL('image/jpeg');
    //Set the "src" tag from iPic to the iCanvasData data.
    $iPic.src = iCanvasData;
    //Remove the 'hide' class from btnOk so it is visible on the screen
    $btnOk.classList.remove('hide');
    //Change the text from btnPicture to "RETAKE PICTURE" so the user knows they can retake the picture 
    //when they reclick the button
    $btnPicture.innerHTML = "RETAKE PICTURE";                   
  }
  
  
  init();
</script>
```

The next step is to add an EventListener to btnOk and btnRemove. 
- When you click on btnOk we remove the camera section from the site. It's no longer needed. We also make sure that the section where our results comes appears.
- When clicked on btnRemove we execute the RunwayML code as seen as above. 

``` javascript
<script>
{
  const init = async() => {
      ...
      $btnPicture.addEventListener('click', process);
      $btnOk.addEventListener('click', nextStep);
      $btnRemove.addEventListener('click', removeBackground);
    
      requestAnimationFrame(draw);
  }
  
  ...
  
  const nextStep = () => {
    $btnOk.classList.add('hide');
    $iCanvasSection.classList.add('hide');
    $oImgSection.classList.remove('hide');
  }
  
  const removeBackground = () => {
    const inputs = {
      "image": iCanvasData,
      "threshold": 0.0001
    };
    
    fetch('http://localhost:8000/query', {
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
    body: JSON.stringify(inputs)
    })
    .then(response => response.JSON())
    .then(outputs => {
      const { image} = outputs;
      $oImg.src = image;
    })
  };
  
  init();
}
</script>
```
For example images, go to my <a href="http://elkedejonckere.wixsite.com/ppproject">blog</a>.
