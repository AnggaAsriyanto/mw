<script>
  import { onMount } from "svelte";

     onMount(() => {
          const cursorCanvas   = document.querySelector('.cursor--canvas');
          const innerCursor    = document.querySelector(".cursor--small");
          const navTop         = document.querySelector('.nav.top');
          const btnMenu        = document.querySelector('.menu');
          const barsMenu       = document.querySelector('.bars-menu');
          const btnLeft        = document.getElementById('btn-left');
          const btnRight       = document.getElementById('btn-right');
          const mpEl           = document.querySelectorAll('.mp');
          const projects       = document.querySelectorAll('.my-project');
          const clicks         = document.querySelectorAll('.click');
          const enters         = document.querySelectorAll('.enter');
          const btnLink        = document.querySelector('.email button');
          const btnLinkMobile  = document.querySelector('.email-m button');
          const myLink         = document.querySelectorAll('.my-link');
          const myLinkMobile   = document.querySelectorAll('.my-link-m');
          const localTime      = document.querySelector('.local-time');
          const idxEl          = document.querySelectorAll('.idx span');

          let clientX         = -100;
          let clientY         = -100;
          let lastX           = 0;
          let lastY           = 0;
          let isStuck         = false;
          let showCursor      = false;
          let colorEl         = 'rgba(0, 0, 0, 0.4)';
          let idxProject      = 0;
          let idxImg          = 0;
          let idxLink         = 0;
          let idxLinkMobile   = 0;
          let shapeBounds     = {
          width: 75,
          height: 75
          };
          let group, stuckX, stuckY, fillOuterCursor;
          let interval = setInterval(() => linkSwipe(), 5000);;


          addActiveClass(projects, 'active');
          showFirst();
          changeIdxEl();

          HideOnMobile();
          window.onresize = HideOnMobile;

          btnMenuClick();

          initCursor();
          initCanvas();
          initHovers();

          mousePseudo(btnLeft);
          mousePseudo(btnRight);

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

          document.addEventListener('keydown', (e) => {
          if(e.keyCode === 13) {
          document.querySelector('.my-project.active').click();
          }
          if(e.keyCode === 37 || e.keyCode === 40) {
          btnLeft.style.opacity = 0;
          idxImg = 0;
          btnLeft.click();
          }
          if(e.keyCode === 39 || e.keyCode === 38) {
          btnRight.style.opacity = 0;
          idxImg = 0;
          btnRight.click();
          }
          })

          document.addEventListener('keyup', (e) => {
          if(e.keyCode === 37 || e.keyCode === 40 || e.keyCode === 39 || e.keyCode === 38) {
          btnLeft.style.opacity = 1;
          btnRight.style.opacity = 1;
          }
          })

          projects.forEach(project => {
          project.addEventListener('click', () => {
          if(idxImg < project.children.length - 1) {
               idxImg++;
          } else {
               idxImg = 0;
          }
          removeShow();
          project.children[idxImg].classList.add('show');
          })
          })


          btnLeft.addEventListener('click', () => {
          if(idxProject > 0) {
          idxProject--;
          } else {
          idxProject = projects.length - 1;
          }
          idxImg = 0;
          removeShow();
          addActiveClass(projects, 'active');
          showFirst();
          changeIdxEl();
          });


          btnRight.addEventListener('click', () => {
          if(idxProject < projects.length - 1) {
          idxProject++;
          } else {
          idxProject = 0;
          }

          idxImg = 0;
          removeShow();
          addActiveClass(projects, 'active');
          showFirst();
          changeIdxEl();
          });

          clicks.forEach(click => {
          click.addEventListener('mouseenter', () => innerCursor.style.animation = 'redClick 1s linear infinite')
          click.addEventListener('mouseleave', () => innerCursor.style.animation = 'none')
          })

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

          function changeIdxEl() {
          idxEl.forEach((index, num) => {
          let zero = (num+1 <= 10) ? 0 : '';
          index.innerText = `${zero}${num + 1}/${zero}${idxEl.length}`;
          })
          }

          function mousePseudo(el) {
          el.addEventListener(`mousedown`, () => {
               el.style.opacity = 0;
          })

          el.addEventListener(`mouseup`, () => {
          el.style.opacity = 1;
          })
          }

          function addActiveClass(el, cls) {
               el.forEach(project => project.classList.remove(cls))
               el[idxProject].classList.add(cls);
          }


          function showFirst() {
          projects[idxProject].children[0].classList.add('show');
          }


          function removeShow() {
          mpEl.forEach(mp => mp.classList.remove('show'))
          }


          function btnMenuClick() {
          let clicked = false;
          btnMenu.addEventListener('click', () => {
          if(!clicked){
               btnMenu.innerText = 'close.';
               barsMenu.classList.add('show');
               clicked = true;
          } else {
               btnMenu.innerHTML = '<i class="fas fa-bars"></>';
               barsMenu.classList.remove('show');     
               clicked = false;
          }
          })
          }


          function HideOnMobile() {
               if(navigator.userAgent.match('Mobile')) {
               innerCursor.style.display = 'none';
               cursorCanvas.style.display = 'none';
               } else {
               innerCursor.style.display = 'block';
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
         <div class="project link enter active">
             <a href="projects">Projects</a>
             <button class="menu"><i class="fas fa-bars"></i></button>
         </div>
         <div class="home link enter">
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
     <div class="main-focus">
         <div class="projects">

             <div class="my-project click">
                 <div class="img-desktop mp">
                     <div class="img-container">
                         <img src="/images/yellcamp_info/yellcamp_desktop.png" alt="yellcamp desktop" class="img-pic">
                         <h4 class="idx"><span></span></h4>
                         <div class="img-title">
                             <h4>Yellcamp</h4>
                         </div>
                     </div>
                 </div>

                 <div class="img-info mp">
                     <h1 class="info-title imgif">Yellcamp</h1>
                     <small class="year imgif">2020</small>
                     <p class="info-description imgif">This is the first website that I created from the web developer course I have completed,  this website is just a prototype to improve backend skills and focuses on campers who want to share and find camping info from around the world.</p>
                     <a class="link-out imgif" href="https://yellcamp.herokuapp.com/">Visit Website <span><i class="fas fa-arrow-up"></i></span></a>
                 </div>

                 <div class="img-mobile mp">
                     <div class="img-container">
                         <div class="img-mobile-1 imc">
                             <img src="/images/yellcamp_info/yellcamp_m1.png" alt="yellcamp mobile" class="img-m1 im">
                         </div>
                         <div class="img-mobile-2 imc">
                             <img src="/images/yellcamp_info/yellcamp_m2.png" alt="yellcamp mobile" class="img-m2 im">
                         </div>
                     </div>
                 </div>
             </div>

             <div class="my-project click">
                 <div class="img-desktop mp" >
                     <div class="img-container">
                         <img src="/images/my_website_info/Angga Asriyanto Desktop.png" alt="" class="img-pic">
                         <h4 class="idx"><span></span></h4>
                         <div class="img-title">
                             <h4 class="min">anggaasriyanto</h4>
                         </div>
                     </div>
                 </div>

                 <div class="img-info mp">
                     <h1 class="info-title imgif">AA</h1>
                     <small class="year imgif">2021</small>
                     <p class="info-description imgif">This is the website you are currently visiting and the first website I created without any course help, at least needed
                         almost three months to perfect it. My website aims to improve my frontend and branding skills as a full-stack developer.</p>
                     <a class="link-out imgif" href="#">Visit Website <span><i class="fas fa-arrow-up"></i></span></a>
                 </div>

                 <div class="img-mobile mp">
                     <div class="img-container">
                         <div class="img-mobile-1 imc">
                             <img src="/images/my_website_info/Angga Asriyanto m1.png" alt="" class="img-m1 im">
                         </div>
                         <div class="img-mobile-2 imc">
                             <img src="/images/my_website_info/Angga Asriyanto m2.png" alt="" class="img-m2 im">
                         </div>
                     </div>
                 </div>
             </div>
     
             <button class="side btn enter" id="btn-left"><i class="fas fa-chevron-left"></i></button>
             <button class="side btn enter" id="btn-right"><i class="fas fa-chevron-right"></i></button>
         </div>
     </div>
 </main>

 <div class="bars-menu">
     <div class="link-box">
         <div class="link-m">
             <a class="about-m fm"  href="about">About</a>
             <a class="projects-m fm"  href="projects">Projects.</a>
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

.nav .active a {
    text-decoration: line-through dashed;
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
    right: 0;
    top: 0;
    text-transform: uppercase;
    background-color: transparent;
    -webkit-user-select: none;
    user-select: none;
}

.nav.bottom {
    width: 100%;
    height: 14%;
    left: 0;
    bottom: 0;
    font-size: 11px;
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

.nav.bottom .link-info .my-link {
    position: absolute;
    opacity: 0;
}

.nav.bottom .link-info .my-link.show {
    transform-origin: bottom;
    animation: showLink 1.5s ease forwards;
    z-index: 1000;
}

.nav.bottom .link-info .my-link.hide {
    transform-origin: top;
    animation: hideLInk 1.5s ease;
    z-index: 0;
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
    background-image: linear-gradient(165deg, rgb(255, 255, 255) 3%, var(--prim-color));
    font-family: var(--f3-family);
    overflow: hidden;
    height: 100vh;
    margin: 0;
    position: relative;
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
    width: 100%;
    height: 100%;
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
    height: 350px;
    writing-mode: vertical-lr;
    display: flex;
    justify-content: flex-end;
    transform: translate(-50%, -50%) scale(-1, -1);
    color: var(--sec-color);
    font-size: 18px;
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
    color: var(--sec-color);
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
    border: none;
    background-color: var(--sec-color);
}

.cursor--canvas {
    width: 100vw;
    height: 100vh;
    z-index: 12000;
}

body, body a, body button {
    cursor: none;
}

.main-focus {
    position: absolute;
    width: 75%;
    height: 70%;
    background-color: transparent;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}

.main-focus .projects {
    position: relative;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

.my-project {
    position: absolute;
    width: 55%;
    height: 70%;
    max-width: 530px;
    max-height: 310px;
    transform: translateY(2em);
    transition: transform 0.9s ease;
    background-color: transparent;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 100;
    &.active {
          transform: translateY(0);
          transition: transform 1s cubic-bezier(0.19, 1, 0.22, 1) 0.7s;
          z-index: 1000;
    }
}

.mp {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 10;
    opacity: 0;
    transition: all 0.8s cubic-bezier(0.19, 1, 0.22, 1);
    &.show {
          z-index: 1000;
          opacity: 1;
          transition: all 1s ease 0.7s;
    }
}

.img-container {
    position: relative;
    width: 100%;
    height: 100%;
}

.img-container .img-pic {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}

.img-container .idx {
    position: absolute;
    top: 2%;
    left: 2%;
    width: 55px;
    height: 55px;
    margin: 0;
    transform: translate(-50%, -50%);
    font-size: 1.5em;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #fff;
    background-color: var(--ter-color);
    box-shadow: 1px 1px 7px rgba(0, 0, 0, 0.1);
    border-radius: 50%;
}

.img-container .idx span {
    font-family: var(--f2-family);
    font-size: 0.6em;
}

.img-container .img-title {
    position: absolute;
    right: 0;
    bottom: 0;
    width: 120px;
    height: 100px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0;
    color: rgb(255, 255, 255);
    letter-spacing: 0.5px;
    font-family: var(--f2-family);
    font-weight: 700;
    overflow: hidden;
}

.img-container .img-title h4 {
    height: 22.5px;
    display: flex;
    justify-content: center;
    align-items: center;
    text-transform: uppercase;
    letter-spacing: 2.5px;
    font-size: 14;
    background-color: var(--ter-color);
    padding: 3px 200px;
    transform: rotateZ(-40deg) translateY(12px) translateX(-30%);
    white-space: nowrap;
    box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
}

.img-container .img-title .min {
    font-size: 9px;
}

.img-desktop.show .img-container .img-title h4 {
    transition: 1.5s ease 0.7s;
    transform: rotateZ(-40deg) translateY(12px) translateX(0%);
}

.img-info {
    height: 100%;
    background-color: transparent;
    font-family: var(--f2-family);
    text-align: center;
    display: flex;
    align-items: center;
    flex-direction: column;
}

.img-info .imgif {
    transform: translateY(60px);
    transition: 1s ease 1s;
}

.img-info.show .imgif {
    transform: translateY(0);
    transition: 1.2s ease;
}


.img-info.show .info-title {
    transition-delay: 0.55s;
}

.img-info.show .year {
    transition-delay: 0.62s;
}

.img-info.show .info-description {
    transition-delay: 0.69s;
}

.img-info.show .link-out {
    transition-delay: 0.76s;
}


.img-info .info-title {
    text-transform: uppercase;
    letter-spacing: 0.5px;
    font-size: 3em;
    margin: 0;
    white-space: nowrap;
}

.img-info .info-title small {
    font-size: 0.5em;
}

.img-info .year {
    font-size: 0.95em;
    font-family: var(--f1-family);
    margin-top: -10px;
}

.img-info .info-description {
    font-size: 0.8em;
    line-height: 1.7em;
    letter-spacing: 0.6px;
    margin: 1em;
    color: rgba(0, 0, 0, 0.9);
}

.img-info .link-out {
    text-decoration: none;
    font-size: 0.7em;
    font-weight: 700;
    text-transform: uppercase;
    
}

.img-info a span {
    display: inline-block;
    transform: rotateZ(45deg);
}

.img-mobile .imc {
    background-color: transparent;
    transition: all 1s ease 0.7s;
}

.img-mobile.show .imc{
    transition: all 1.2s ease 0.7s;
}

.imc {
    position: absolute;
    width: 180px;
    height: 100%;
    transform: translate(-50%, -50%);
    box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.1);
}

.imc > img {
    width: 100%;
    height: 100%;
}

.img-mobile-1 {
    top: 35%;
    left: 30%;
}

.img-mobile.show .img-mobile-1 {
    top: 40%;
    left: 35%;
}

.img-mobile-2 {
    top: 65%;
    left: 70%;
    box-shadow: 3px 3px 10px rgba(0, 0, 0, 0.1),
                -1px -1px 10px rgba(0, 0, 0, 0.1);
}

.img-mobile.show .img-mobile-2 {
    top: 60%;
    left: 65%;
}

.side {
    position: absolute;
    border: none;
    background-color: transparent;
    font-size: 1.4em;
    color: var(--ter-color);
    visibility: hidden;
    transform: translate(-50%, -50%) rotateZ(-45deg);
    z-index: 2000;
}

.side#btn-right {
    top: 7.5%;
    left: 83%; 
    animation: showRightBtn 1.7s ease 0.5s forwards;
}

.side#btn-left {
    top: 92.5%;
    left: 17%; 
    animation: showLeftBtn 1.7s ease 0.5s forwards;
}

.side#btn-right:focus, .side#btn-left:focus {
    outline: none;
}

.side#btn-left:active, .side#btn-right:active {
    color: rgba(0, 0, 0, 0);
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

@keyframes hideLInk {
    from {
        transform: rotateX(0) translateY(0);
        opacity: 1;
    }

    to {
       transform: rotateX(90deg) translateY(-10px);
       opacity: 0;
    }
    
}

@keyframes showRightBtn {
    0% {
        top: 50%;
        left: 50%; 
    }

    50% {
        visibility: hidden;
    }

    100% {
        top: var(--btn-right-top);
        left: var(--btn-right-left);
        visibility: visible;
    }
}

@keyframes showLeftBtn {
    0% {
        top: 50%;
        left: 50%; 
        opacity: 0;
    }

    50% {
        visibility: hidden;
    }

    100% {
        top: var(--btn-left-top);
        left: var(--btn-left-left); 
        visibility: visible;
    }
}

@keyframes redClick {
    0% {
        background-color: transparent;
    }

    50% {
        background-color: rgb(255, 0, 0);
    }

    100% {
        background-color: transparent;
    }
}

@media(max-width: 565px) {
    :root {
        --btn-right-top: 10%;
        --btn-right-left: 47%;
        --btn-left-top: 87%;
        --btn-left-left: 47%;
    }

    .hom, .nav.bottom, .nav.top a  {
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


    .my-project {
        width: 100%;
        height: 35%;
        max-width: 280px;
        max-height: 170px;
    }

    .img-info {
        transform: scale(0.85);
    }

    .img-info .info-description {
        width: 150%;
        margin: 1.2em;
    }

    .imc {
        width: 50%;
        height: 150%;
    }

    .side {
        font-size: 1em;
        transform: rotateZ(-90deg);
    }
}

@media(max-width: 380px) {
    .link-m {
        transform: scale(0.8);
        left: -2%;
        top: 78%;
    }

    .img-info {
        transform: scale(0.8);
    }

    .img-container .idx {
        transform: translate(-50%, -50%) scale(0.9);
    }

    .email-m {
        transform: translate(-50%, -50%) scale(-1, -1) scale(0.8);
    }
}

@media(max-width: 300px) {
    .my-project {
        width: 100%;
        height: 35%;
        max-width: 280px;
        max-height: 150px;
    }

    .img-info {
        transform: scale(0.6);
    }

    .img-info .info-description {
        width: 250%;
        margin: 1.2em;
    }
}

@media only screen and (device-width: 768px) {
    :root {
        --btn-right-top: 20%;
        --btn-left-top: 80%;
    }

    .my-project {
        width: 55%;
        height: 70%;
        max-width: 530px;
        max-height: 200px;
    }

    .img-info {
        transform: scale(0.85);
    }

    .img-info .info-description {
        width: 150%;
        margin: 1.2em;
    }

    .imc {
        width: 60%;
        height: 150%;
    }
}

@media only screen and (device-width: 1024px) and (device-height: 1366px) {
    :root {
        --btn-right-top: 20%;
        --btn-left-top: 80%;
    }

    .my-project {
        width: 55%;
        height: 70%;
        max-width: 530px;
        max-height: 260px;
    }

    .img-info .info-description {
        width: 150%;
        margin: 1.2em;
    }

    .imc {
        width: 60%;
        height: 150%;
    }
}

@media(max-height: 450px) {
    .img-info {
        transform: scale(0.6);
    }

    .img-info .info-description {
        width: 150%;
    }

    .imc {
        width: 40%;
        height: 95%;
    }
}
 </style>