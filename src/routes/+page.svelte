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
                 <a class="my-link" href="mailto:halo@gmail.com">Linkedin</a>
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