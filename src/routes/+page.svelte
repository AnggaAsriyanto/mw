<script>
     import { onMount } from "svelte";

     const name = 'Angga Asriyanto';
     const years = '2020 - 2023';
     const jobs = 'web developer'

     onMount(() => {
          const cursorCanvas   = document.querySelector('.cursor--canvas');
          const cursorSmall    = document.querySelector('.cursor--small');
          const navTop         = document.querySelector('.nav.top');
          const btnMenu        = document.querySelector('.menu');
          const barsMenu       = document.querySelector('.bars-menu');
          const circle         = document.querySelector('.circle');
          const innerCursor    = document.querySelector(".cursor--small");
          const btnLink        = document.querySelector('.email button');
          const btnLinkMobile  = document.querySelector('.email-m button');
          const myLink         = document.querySelectorAll('.my-link');
          const myLinkMobile   = document.querySelectorAll('.my-link-m');
          const localTime      = document.querySelector('.local-time');

          let clientX         = -100;
          let clientY         = -100;
          let lastX           = 0;
          let lastY           = 0;
          let isStuck         = false;
          let showCursor      = false;
          let idxLink         = 0;
          let idxLinkMobile   = 0;
          let shapeBounds = {
               width: 75,
               height: 75
          };
          let group, stuckX, stuckY, fillOuterCursor;
          let colorEl  = 'rgba(255, 255, 255, 0.6)';
          let interval = setInterval(() => linkSwipe(), 5000);;


          HideOnMobile();
          window.onresize = HideOnMobile;

          initCursor();
          initCanvas();
          initHovers();

          btnMenuClick();

          btnLink.addEventListener('click', () => {
               clearInterval(interval);
               linkSwipe();
               interval = setInterval(() => linkSwipe(), 5000)
          });

          btnLinkMobile.addEventListener('click', () => {
               const removeShow = () => myLinkMobile.forEach(link => link.classList.remove('show'));

               if(idxLinkMobile < myLinkMobile.length - 1) {
                    idxLinkMobile++;
               } else {
                    idxLinkMobile = 0;
               }

               removeShow();
                    myLinkMobile[idxLinkMobile].classList.add('show');
          });

          generateLocalTime();
          setInterval(generateLocalTime, 10000);

          async function generateLocalTime() {
               const res   = await fetch('https://timezoneapi.io/api/timezone/?Asia/Jakarta&token=aQoaYBMGgzWbMHFsrmFw');
               const data  = await res.json();
               let day     = data.data.datetime.day_abbr;
               let hours   = data.data.datetime.time.slice(0, 2);
               let minutes = data.data.datetime.time.slice(3, 5);
               let amPm    = data.data.datetime.hour_am_pm;

               if(hours >= 12) {
                    hours = hours - 12;
               }

               localTime.innerHTML = `Local time: ${hours}:${minutes} (${amPm}) - ${day}`;
          }

          function linkSwipe() {
               const removeShowHide = () => {
                    myLink.forEach(link => {
                         link.classList.remove('hide');
                         link.classList.remove('show');
                    })
               }
               if(idxLink <myLink.length - 1) {
                    removeShowHide();
                    myLink[idxLink].classList.add('hide');
                    myLink[idxLink + 1].classList.add('show');
                    idxLink++;
               } else {
                    removeShowHide();
                    myLink[idxLink].classList.add('hide');
                         idxLink = 0;
                    myLink[0].classList.add('show');
               }
          }

          function btnMenuClick() {
               let clicked = false;
               btnMenu.addEventListener('click', () => {
                    if(!clicked){
                         // document.body.style.overflowY = 'hidden';
                         btnMenu.innerText = 'Close.';
                         barsMenu.classList.add('show');
                         clicked = true;
                    } else {
                         // document.body.style.overflowY = 'scroll';
                         btnMenu.innerHTML = '<i class="fas fa-bars"></>';
                         barsMenu.classList.remove('show');     
                         clicked = false;
                    }
               })
          }

          function HideOnMobile() {
               if(navigator.userAgent.match('Mobile')) {
                    cursorSmall.style.display = 'none';
                    cursorCanvas.style.display = 'none';
               } else {
                    cursorSmall.style.display = 'block';
                    cursorCanvas.style.display = 'block';
               }
          }


          function initCursor() {
               document.addEventListener("mousemove", e => {
                    clientX = e.clientX;
                    clientY = e.clientY;
               });
               const render = () => {
               innerCursor.style.transform = `translate(${clientX}px, ${clientY}px)`;
                    requestAnimationFrame(render);
               };
               requestAnimationFrame(render);
          };

          function initCanvas() {
               const canvas = document.querySelector(".cursor--canvas");
               paper.setup(canvas);
               const strokeColor = colorEl;
               const strokeWidth = 1;
               const segments = 8;
               const radius = 15;
               
               const noiseScale = 150; // speed
               const noiseRange = 4; // range of distortion
               let isNoisy = false; // state
               
               const polygon = new paper.Path.RegularPolygon(
               new paper.Point(0, 0),
               segments,
               radius
          );

          polygon.strokeColor = strokeColor;
          polygon.strokeWidth = strokeWidth;
          polygon.smooth();
          group = new paper.Group([polygon]);
          group.applyMatrix = false;
          
          const noiseObjects = polygon.segments.map(() => new SimplexNoise());
          let bigCoordinates = [];
          
          // function for linear interpolation of values
          const lerp = (a, b, n) => {
               return (1 - n) * a + n * b;
          };
          
          // function to map a value from one range to another range
          const map = (value, in_min, in_max, out_min, out_max) => {
               return (
                    ((value - in_min) * (out_max - out_min)) / (in_max - in_min) + out_min
               );
          };

          paper.view.onFrame = event => {
               if (!isStuck) {
                    lastX = lerp(lastX, clientX, 0.2);
                    lastY = lerp(lastY, clientY, 0.2);
                    group.position = new paper.Point(lastX, lastY);
               } else if (isStuck) {
                    lastX = lerp(lastX, stuckX, 0.2);
                    lastY = lerp(lastY, stuckY, 0.2);
                    group.position = new paper.Point(lastX, lastY);
               }
               
               if (isStuck && polygon.bounds.width < shapeBounds.width) { 
                    polygon.scale(1.08);
               } else if (!isStuck && polygon.bounds.width > 30) {
                    if (isNoisy) {
                         polygon.segments.forEach((segment, i) => {
                         segment.point.set(bigCoordinates[i][0], bigCoordinates[i][1]);
                    });
                    isNoisy = false;
                    bigCoordinates = [];
                    }
                    const scaleDown = 0.92;
                    polygon.scale(scaleDown);
               }
               
               if (isStuck && polygon.bounds.width >= shapeBounds.width) {
                    isNoisy = true;
                    if (bigCoordinates.length === 0) {
                    polygon.segments.forEach((segment, i) => {
                         bigCoordinates[i] = [segment.point.x, segment.point.y];
                    });
                    }
                    
                    // loop over all points of the polygon
                    polygon.segments.forEach((segment, i) => {
                    
                    const noiseX = noiseObjects[i].noise2D(event.count / noiseScale, 0);
                    const noiseY = noiseObjects[i].noise2D(event.count / noiseScale, 1);
                    
                    const distortionX = map(noiseX, -1, 1, -noiseRange, noiseRange);
                    const distortionY = map(noiseY, -1, 1, -noiseRange, noiseRange);
                    
                    const newX = bigCoordinates[i][0] + distortionX;
                    const newY = bigCoordinates[i][1] + distortionY;
                    
                    segment.point.set(newX, newY);
                    });
                    
               }
               polygon.smooth();
               };
          }

          function initHovers() {

               function forEnter(e) {
                    const navItem = e.currentTarget;
                    const navItemBox = navItem.getBoundingClientRect();
                    stuckX = Math.round(navItemBox.left + navItemBox.width / 2);
                    stuckY = Math.round(navItemBox.top + navItemBox.height / 2);
                    isStuck = true;
                    innerCursor.style.display = 'none';
               }

               function forLeave() {
                    isStuck = false;
                    innerCursor.style.display = 'block';
               }

               const handleMouseEnter = e => {
                    forEnter(e);
                    shapeBounds = {
                    width: 75,
                    height: 75
                    }
               };

               const handleMouseLeave = () => {
                    forLeave();
               };

               const handleMouseEnterBtn = e => {
                    forEnter(e);
                    shapeBounds = {
                    width: 35,
                    height: 35
                    }
               };

               const handleMouseLeaveBtn = () => {
                    forLeave();
               };


               const linkItems = document.querySelectorAll(".link");
               linkItems.forEach(item => {
                    item.addEventListener("mouseenter", handleMouseEnter);
                    item.addEventListener("mouseleave", handleMouseLeave);
               });

               const btnItems = document.querySelectorAll(".btn");
               btnItems.forEach(btn => {
                    btn.addEventListener("mouseenter", handleMouseEnterBtn);
                    btn.addEventListener("mouseleave", handleMouseLeaveBtn);
               })
          };
     })
</script>
<div class="navbars">   
     <nav class="nav top">
         <div class="project link enter">
             <a href="projects">Projects.</a>
             <button class="menu"><i class="fas fa-bars"></i></button>
         </div>
         <div class="home link enter active">
             <a id="home" href="/">AA.</a>
         </div>
         <div class="about link enter">
             <a href="about">About.</a>
         </div>
     </nav>

     <nav class="nav bottom">
         <div class="email enter">
             <button class="btn-click btn"><i class="fas fa-arrow-circle-up"></i></button>
             <div class="link-info enter">
                 <a class="my-link show" href="mailto:anggazxcasd@gmail.com">anggazxcasd@gmail.com</a>
                 <a class="my-link hide" href="mailto:halo@gmail.com">Linkedin</a>
             </div>
         </div>
         <p class="local-time enter" href="#">Local time: 0:00 (am) - Mon</p>
     </nav>
</div>

<main>
     <div>
         <svg class="waves" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
         viewBox="0 24 150 28" preserveAspectRatio="none" shape-rendering="auto">
         <defs>
         <path id="gentle-wave" d="M-160 44c30 0 58-18 88-18s 58 18 88 18 58-18 88-18 58 18 88 18 v44h-352z" />
         </defs>
         <g class="parallax">
         <use xlink:href="#gentle-wave" x="48" y="0" fill="rgba(255,255,255,0.7)" />
         <use xlink:href="#gentle-wave" x="48" y="3" fill="rgba(255,255,255,0.5)" />
         <use xlink:href="#gentle-wave" x="48" y="5" fill="rgba(255,255,255,0.3)" />
         <use xlink:href="#gentle-wave" x="48" y="7" fill="#fff" />
         </g>
         </svg>
     </div>

     <div class="intro">
         <h3 class="name me">
            <span>Angga</span> Asriyanto
         </h3>
         <h1 class="prof me">
             Web Developer.
         </h1>
         <h1 class="year me">
             2020 - <span>2023</span>
         </h1>
     </div>
     
     <div class="main-focus center">
         <div class="idea">
             <h1>difference</h1>
             <h1>planning imagination</h1>
             <h1>trend creativity</h1>
             <h1>responsive inspiration</h1>
             <h1>fresh innovation</h1>
             <h1>hierarchy magnificence</h1>
             <h1>modern minimalist</h1>
             <h1>experience</h1>
         </div>
     </div>

     <div class="text-container center;">
         <svg class="circle" viewBox="0 0 200 200">
             <path id="curve" d="
             M 25, 100
             a 25,25 0 1,1 150,0
             a 25,25 0 1,1 -150,0
             " />
             <text class="text">
               <textPath xlink:href="#curve" class="text-outline">
                 { name } • { years } • { jobs }
                 • Indonesia • • • • • • • • • • • • • • • • • • • • •
                 • • • • • • • • • • • • • • • •
               </textPath>
             </text>
           </svg>
     </div>
</main> 

<div class="bars-menu">
     <div class="link-box">
         <div class="link-m">
             <a class="about-m fm"  href="about">About</a>
             <a class="projects-m fm"  href="projects">Projects</a>
         </div>

         <div class="email-m">
             <div class="link-info-m"> 
                 <a class="fm my-link-m show" href="mailto:anggazxcasd@gmail.com">anggazxcasd<span>@</span>gmail.com</a>
                 <a class="my-link-m fm" href="mailto:halo@gmail.com">Linkedin</a>
             </div>
             <button class="btn-click"><i class="fas fa-arrow-circle-down"></i></button>
         </div>
     </div>
</div>

<div class="cursor cursor--small"></div>
<canvas class="cursor cursor--canvas" resize></canvas>

<style>
.nav {
    position: relative;
    display: flex;
    position: fixed;
    align-items: flex-end;
    justify-content: space-between;
    align-items: center;
    z-index: 1000;
    padding: 0 5%;
}

.nav.top .home {
    position: absolute;
    font-family: var(--f1-family);
    left: 50%;
    transform: translate(-50%, 0);
}

.nav a {
    display: flex;
    text-decoration: none;
    color: var(--sec-color);
    display: inline-block;
    font-size: 11px;
    font-weight: 400;
    letter-spacing: 1.8px;
    margin: 0 20px;
}

.nav.bottom p {
    color: var(--sec-color);
    font-size: 11px;
    letter-spacing: 1.8px;
    margin: 0 20px;
}

.nav a:hover {
    font-weight: 700;
}

.nav a:active, .fm:active {
    color: rgba(255, 255, 255, 0);
}

.nav.top {
    width: 100%; 
    height: 14%;
    left: 0; 
    top: 0;
    text-transform: uppercase;
    background-color: transparent;
    -webkit-user-select: none;
    user-select: none;
    transform: translateY(-100%);
    animation: showNav 1.8s ease 5.5s forwards;
}

.nav.bottom {
    width: 100%; 
    height: 14%;
    left: 0; 
    bottom: 0;
    font-size: 11px;
    transform: translateY(100%);
    font-family: var(--f3-family);
    animation: showNav 1.8s ease 5.5s forwards;
}

.nav.bottom .email {
    width: 210px;
    display: flex;
    justify-content: flex-start;
}

.nav.bottom .email button {
    background-color: transparent;
    border: none;
    font-size: 10px;
    transform: translate(4px, 0px);
    color: var(--sec-color)
}

.nav.bottom .link-info {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
}

.nav.bottom .my-link {
    position: absolute;
    opacity: 0;
}

.nav.bottom .my-link.show {
    transform-origin: bottom;
    z-index: 1000;
    animation: showLink 1.5s ease forwards;
}

.nav.bottom .my-link.hide {
    transform-origin: top;
    z-index: 0;
    animation: hideLink 1.5s ease;
}

.btn-click:active {
    opacity: 0;
}

.nav.bottom .email a:hover {
    text-decoration: underline;
}

.nav .home a {
    font-size: 35px;
    font-weight: 700;
    letter-spacing: 0;
    display: block;
}

.menu {
    display: none;
    color: var(--sec-color);
}

main {
    background-image: linear-gradient(to bottom, rgb(193, 255, 236) -30%, var(--prim-color));
    position: relative;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
}

.bars-menu {
    position: absolute;
    height: 100%; 
    width: 100%;
    top: 0;
    display: none;
    transform: translateX(-100vw);
    z-index: 100;
    transition: 1s ease;
    background-color: rgb(70, 68, 68);
    & .link-m a {
        opacity: 0;
        transform: translateX(-10vw);
        transition: 1s ease;
    }
    &.show {
        transform: translateX(0);
        & .link-m a {
            opacity: 1;
            transform: translateX(0);
            transition: transform 1s ease,
                        opacity 1s ease;
            &:nth-of-type(1) {
                transition-delay: 0.4s;
            }
            &:nth-of-type(2) {
                transition-delay: 0.5s;
            }
        }
    }
}

.link-box {
    position: relative;
    width: 100%; height: 100%;
}

.link-m {
    position: absolute;
    top: 80%; 
    left: 6%;
    display: flex;
    flex-direction: column;
}

.link-m a {
    margin: -11px 0;
}

.fm {
    font-size: 55px;
    letter-spacing: 0.5px;
    text-transform: uppercase;
    text-decoration: none;
    color: var(--sec-color);
}

.email-m {
    position: absolute;
    top: 47%; 
    left: 90%;
    display: flex;
    justify-content: flex-end;
    writing-mode: vertical-lr;
    height: 350px;
    color: var(--sec-color);
    font-size: 18px;
    transform: translate(-50%, -50%) scale(-1, -1);
}

.email-m span {
    font-weight: 700;
}

.link-info-m {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: flex-end;
}

.link-info-m .my-link-m {
    position: absolute;
    color: rgba(255, 255, 255, 0.6);
    text-decoration: none;
    opacity: 0;
    z-index: 100;
    transform: translateY(-10px);
}

.link-info-m .my-link-m.show {
    opacity: 1;
    z-index: 1000;
    transform: translateY(0);
    transition: all 1.5s ease;
}

.cursor {
    position: fixed;
    left: 0; 
    top: 0;
    pointer-events: none;
  }

.cursor--small {
    width: 5px;
    height: 5px;
    left: -2.5px;
    top: -2.5px;
    border-radius: 50%;
    z-index: 11000;
    background: var(--sec-color);
}

.cursor--canvas {
    width: 100vw;
    height: 100vh;
    z-index: 12000;
}

body, body a, body button {
    cursor: none;
}

.intro {
    position: absolute;
    color: var(--sec-color);
    font-family: var(--f2-family);
    top: 50%;
    left: 50%;
    display: flex;
    justify-content: center;
    flex-direction: column;
    transform: translate(-50%, -50%);
}

.intro .me {
    margin: 0;
    text-align: center;
}

.intro .name {
    color:  var(--sec-color);
    padding: 5px 0;
    letter-spacing: 2px;
    font-size: 20px;
    font-weight: 200;
    text-transform: uppercase;
    transform: translateY(50px);
    -moz-animation: goUp 1s ease 1.5s forwards,
                    goLeft 1s ease 3.5s forwards;
    -webkit-animation: goUp 1s ease 1.5s forwards,
                       goLeft 1s ease 3.5s forwards;
    animation: goUp 1s ease 1.5s forwards,
               goLeft 1s ease 3.5s forwards;
}

.intro .name span {
    font-size: 0;
    font-weight: 700;
    color: var(--sec-color);
    -moz-animation: popup 1s cubic-bezier(0.96, -0.03, 0.26, 0.9) forwards 0.3s;
    -webkit-animation: popup 1s cubic-bezier(0.96, -0.03, 0.26, 0.9) forwards 0.3s;
    animation: popup 1s cubic-bezier(0.96, -0.03, 0.26, 0.9) forwards 0.3s;
}

.intro .prof {
    opacity: 0;
    font-size: 55px;
    -moz-animation: showProf 1s cubic-bezier(0.4, 0, 1, 1) forwards 1.7s,
                    dismall 1.5s cubic-bezier(0.96, -0.03, 0.26, 0.9) 3.8s forwards;
    -webkit-animation: showProf 1s cubic-bezier(0.4, 0, 1, 1) forwards 1.7s,
                       dismall 1.5s cubic-bezier(0.96, -0.03, 0.26, 0.9) 3.8s forwards;
    animation: showProf 1s cubic-bezier(0.4, 0, 1, 1) forwards 1.7s,
               dismall 1.5s cubic-bezier(0.96, -0.03, 0.26, 0.9) 3.8s forwards;
}


.intro .year {
    font-size: 14px;
    font-weight: 200;
    padding: 15px 0;
    letter-spacing: 3px;
    transform: translateY(-50px);
    opacity: 0;
    -moz-animation: goDown 1s ease-out forwards 1.5s,
                    goRight 1s ease forwards 3.5s;
    -webkit-animation: goDown 1s ease-out forwards 1.5s,
                    goRight 1s ease forwards 3.5s;
    animation: goDown 1s ease-out forwards 1.5s,
               goRight 1s ease forwards 3.5s;
}

.intro .year span {
    font-weight: 700;
}

.main-focus {
    position: relative;
    top: 50%;
    left: 50%;
    height: var(--mainFocus-size);
    width: var(--mainFocus-size);
    transform: translate(-50%, -50%);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-transform: uppercase;
    font-family: var(--f3-family);
    background-color: transparent;
    border-radius: 50%;
    color: var(--sec-color);
    z-index: 50;
    overflow: hidden;
}

.text-container {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%) scale(0.5);
    width: var(--textCont-size);
    height: var(--textCont-size);
    background-color: transparent;
    z-index: 20;
    opacity: 0;
    -moz-animation: showOutline 2.5s ease 4.7s forwards;
    -webkit-animation: showOutline 2.5s ease 4.7s forwards;
    animation: showOutline 2.5s ease 4.7s forwards;
}

#curve {
    fill: transparent;
}

.text-outline {
    font-size: 5px;
    font-family: var(--f2-family);
    font-weight: 400;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    fill: rgb(255, 238, 0);
}

.idea {
    opacity: 0;
    width: 200%;
    height: 100%;
    display: flex;
    align-items: center;
    flex-direction: column;
    transform: rotateZ(-28deg) translateY(-10px);
    -moz-animation: showFocus 1s ease 4.5s forwards;
    -webkit-animation: showFocus 1s ease 4.5s forwards;
    animation: showFocus 1s ease 4.5s forwards;
}

.idea h1 {
    word-spacing: 8px;
    margin: 0 auto;
    letter-spacing: 0.5px;
    font-size: 30px;
    font-weight: 700;
}

.idea h1:nth-of-type(odd) {
    transform: translateX(100%);
    -moz-animation: slideLeftRight 3s cubic-bezier(0.075, 0.82, 0.165, 1) 5.1s forwards;
    -webkit-animation: slideLeftRight 3s cubic-bezier(0.075, 0.82, 0.165, 1) 5.1s forwards;
    animation: slideLeftRight 3s cubic-bezier(0.075, 0.82, 0.165, 1) 5.1s forwards;
}

.idea h1:nth-of-type(even) {
    transform: translateX(-100%);
    -moz-animation: slideLeftRight 3s cubic-bezier(0.075, 0.82, 0.165, 1) 5.1s forwards;
    -webkit-animation: slideLeftRight 3s cubic-bezier(0.075, 0.82, 0.165, 1) 5.1s forwards;
    animation: slideLeftRight 3s cubic-bezier(0.075, 0.82, 0.165, 1) 5.1s forwards;
}

.waves {
    position: absolute;
    width: 100%;
    height: 10%;
    transform: scale(-1, -1) translateY(100%);
    -moz-animation: showWave 5s ease 3.8s forwards;
    -webkit-animation: showWave 5s ease 3.8s forwards;
    animation: showWave 5s ease 3.8s forwards;
    z-index: 500;
}

.parallax > use {
    animation: move-forever 25s cubic-bezier(.55,.5,.45,.5) infinite;
}
.parallax > use:nth-child(1) {
    animation-delay: -2s;
    animation-duration: 7s;
}
.parallax > use:nth-child(2) {
    animation-delay: -3s;
    animation-duration: 10s;
}
.parallax > use:nth-child(3) {
    animation-delay: -4s;
    animation-duration: 13s;
}
.parallax > use:nth-child(4) {
    animation-delay: -5s;
    animation-duration: 20s;
}

@keyframes showLink {
    from {
        transform: rotateX(90deg) translateY(10px);
        opacity: 0;
    }

    to {
       transform: rotateX(0) translateY(0);
       opacity: 1;
    }
    
}

@keyframes hideLink {
    from {
        transform: rotateX(0) translateY(0);
        opacity: 1;
    }

    to {
       transform: rotateX(90deg) translateY(-10px);
       opacity: 0;
    }
    
}

@keyframes showWave {
    to {transform: scale(-1, -1) translateY(0)}
}

@keyframes move-forever {
    0% {transform: translate3d(-90px,0,0)}
    100% {transform: translate3d(85px,0,0)}
}

@keyframes rotateForever {
    from {transform: translate(-50%, -50%) rotateZ(28deg)}
    to {transform: translate(-50%, -50%) rotateZ(388deg)}
}

@keyframes showNav {
    to {transform: translateY(0)}
}

@keyframes showOutline {
    0% {
        opacity: 1;
        transform: translate(-50%, -50%) rotateZ(-10deg) scale(0.8);
    }

    100% {
        opacity: 1;
        transform: translate(-50%, -50%) rotateZ(28deg);
    }
}

@keyframes slideLeftRight {
    to {transform: translateX(0)}
}


@keyframes showFocus {
    0% {opacity: 0}
    100% {opacity: 1}
}

@keyframes popup {
    0% {font-size: 0}
    100% {font-size: 20px}
}

@keyframes dismall {
    0% {font-size: 55px}
    100% {font-size: 0}
}

@keyframes goUp {
    0% {transform: translateY(50px)}
    100% {transform: translateY(0)}
}

@keyframes showProf {
    0% {opacity: 0}
    100% {opacity: 1}
}

@keyframes goDown {
    0% {
        transform: translateY(-50px);
        opacity: 0;
    }

    100% {
        transform: translateY(0);
        opacity: 1;
    }
}

@keyframes goRight {
    0% {transform: translateX(0)}

    100% {
        transform: translateX(150px);
        opacity: 0;
        display: none;
    }
}

@keyframes goLeft {
    0% {transform: translateX(0)}

    100% {
        transform: translateX(-150px);
        opacity: 0;
        display: none;
    }
}

@media(max-width: 565px) {
    .intro {
        transform: translate(-50%, -50%) scale(0.9);
    }

    .hom, .nav.bottom, .nav.top a {
        display: none;
    }

    .cursor--small {
        opacity: 0;
    }

    .nav .home a {
        display: block;
    }

    .menu {
        display: flex;
        text-align: center;
        font-size: 12px;
        background-color: transparent;
        border: none;
        text-transform: uppercase;
        letter-spacing: 1.8px;
        transform: translateX(7px);
    }

    .bars-menu {
        transition: transform 0.8s ease;
        display: block;
    }

    .bars-menu .email-m button {
        background-color: transparent;
        border: none;
        margin-top: 2em;
        font-size: 1em;
        color: var(--sec-color);
    }

    .bars-menu .email-m a {
        font-size: 18px;
        text-transform: none;
    }

    .nav a:hover {
        font-weight: 400;
    }
}

@media(max-width: 380px) {
    :root {
        --mainFocus-size: 255px;
        --textCont-size: 382.5px;
    }
    .intro {
        transform: translate(-50%, -50%) scale(0.8);
    }

    .link-m {
        transform: scale(0.8);
        left: -2%;
        top: 78%;
    }

    .email-m {
        transform: translate(-50%, -50%) scale(-1, -1) scale(0.8);
    }
}

@media(max-height: 450px) {
    :root {
        --mainFocus-size: 235px;
        --textCont-size: 345.5px;
    }
}

@media(max-height: 325px) {
    :root {
        --mainFocus-size: 205px;
        --textCont-size: 310.5px;
    }
}
</style>