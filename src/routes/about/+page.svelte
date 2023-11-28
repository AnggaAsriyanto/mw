<script>
    import { onMount } from "svelte";

    onMount(() => {
        const cursorCanvas   = document.querySelector('.cursor--canvas');
        const innerCursor    = document.querySelector(".cursor--small");
        const navTop         = document.querySelector('.nav.top');
        const btnMenu        = document.querySelector('.menu');
        const barsMenu       = document.querySelector('.bars-menu');
        const hello          = document.querySelector('.hello h1');
        const helloDiv       = document.querySelector('.hello');
        const hide           = document.querySelector('.hide-eyes');
        const homeEl         = document.getElementById('home');
        const btnLink        = document.querySelector('.email button');
        const btnLinkMobile  = document.querySelector('.email-m button');
        const myLink         = document.querySelectorAll('.my-link');
        const myLinkMobile   = document.querySelectorAll('.my-link-m');
        const localTime      = document.querySelector('.local-time');
        const photoCont      = document.querySelector('.photo-container');
        const main           = document.querySelector('main')

        let clientX         = -100;
        let clientY         = -100;
        let lastX           = 0;
        let lastY           = 0;
        let isStuck         = false;
        let colorEl         = 'rgba(0, 0, 0, 0.4)';
        let idxLink         = 0;
        let idxLinkMobile   = 0;
        let shapeBounds     = {
        width: 75,
        height: 75
        };
        let group, stuckX, stuckY, fillOuterCursor;
        let interval = setInterval(() => linkSwipe(), 5000);

        HideOnMobile();
        window.onresize = HideOnMobile;

        btnMenuClick();

        initCursor();
        initCanvas();
        initHovers();

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


        hello.innerHTML = hello.innerText
                        .split('')
                        .map((letter, idx) => `<span style="transition-delay:${idx * 90}ms">${letter}</span>`)
                        .join('');

                        
        main.addEventListener('scroll', () => {
            const triggerBottom = window.innerHeight / 4.7 * 4;
            var helloTop = hello.getBoundingClientRect().top;

            if(Math.round(0.10 *helloTop) < Math.round(0.10*triggerBottom)) {
                // hide.style.animation = 'hideStyle 1s ease';
                hide.classList.add('animating');
                helloDiv.classList.add('show');
                photoCont.classList.add('right');
            } else {
                // hide.style.animation = 'none';
                hide.classList.remove('animating');
                helloDiv.classList.remove('show');
                photoCont.classList.remove('right');
            }
        })

        function HideOnMobile() {
            if(navigator.userAgent.match('Mobile')) {
                innerCursor.style.display = 'none';
                cursorCanvas.style.display = 'none';
            } else {
                innerCursor.style.display = 'block';
                cursorCanvas.style.display = 'block';
            }
        }

        function btnMenuClick() {
            let clicked = false;
            btnMenu.addEventListener('click', () => {
            if(!clicked){
                document.body.style.overflowY = 'hidden';
                btnMenu.innerHTML = 'close.';
                barsMenu.classList.add('show');
                clicked = true;
            } else {
                document.body.style.overflowY = 'scroll';
                btnMenu.innerHTML = '<i class="fas fa-bars"></>';
                barsMenu.classList.remove('show');     
                clicked = false;
            }
            })
        }
        
        
        function initCursor() {
            document.addEventListener("mousemove", e => {
                clientX = e.clientX;
                clientY = e.clientY;
            }
        );
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
        <div class="home link enter">
            <a id="home" href="/">AA.</a>
        </div>
        <div class="about link enter active">
            <a href="about">About</a>
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
        <div class="page">
            <div class="photo-container">
                <div class="photo">
                    <div class="hide-eyes"></div>
                    <div class="after pp"></div>
                    <img class="pp" src="Angga adjus.png" alt="Angga Asriyanto">
                </div>

                <div class="name">ANGGA</div>
            </div>

            <div class="heading">
                <h1><span>Full-Stack</span> <span>Web Developer</span> <span>from Indonesia</span></h1>
            </div>
        </div>
        <div class="page">
            <div class="hello info-about">
                <h1>Hello!</h1>

                <div class="text">
                    <p>
                        My name is Angga Asriyanto. I am a full-stack web developer from Indonesia. Creating a website that works well and has a beautiful design is my responsibility, goal, and pleasure. For me, showing something that we have, be it art, goods, company or the person themselves to the world through a website is something to be proud of.                        </p>

                    <!-- <p>
                        My 1+ year of experience in this field has developed me to work more efficiently, quickly, and easily in dealing with problems. 
                        The web technologies that I master include HTML, CSS, javascript, node.js, MongoDB, Postman, Git and Github. I also learned python language and understand a little bit about data science.
                    </p> -->
                </div>
            </div>
        </div>
    </div>
</main>

<div class="bars-menu">
    <div class="link-box">
        <div class="link-m">
            <a class="about-m fm"  href="about">About.</a>
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

<style lang="css">
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
    color: var(--sec-color-dark);
    display: inline-block;
    font-size: 11px;
    font-weight: 400;
    letter-spacing: 1.8px;
    margin: 0 20px;
}

.nav.bottom p {
    color: var(--sec-color-dark);
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
    color: var(--sec-color-dark)
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
    animation: hideLink 1.5s ease;
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
    background-color: var(--prim-color);
    display: none;
    color: var(--sec-color-dark);
}

main {
    height: 100vh;
    margin: 0;
    overflow-x: hidden;
    scroll-behavior: smooth;
    scroll-snap-type: y mandatory;
}

.main-focus .page {
    position: relative;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    scroll-snap-align: center;
}

.page .heading {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: var(--fix-topLeft);
    z-index: 100;
    width: 800px;
    height: 140px;
    display: flex;
    align-items: center;
    overflow: hidden;
}

.page .heading h1 {
    font-size: 3em;
    text-align: center;
    letter-spacing: 2px;
    color:rgba(0, 0, 0, 0.6);
    text-transform: uppercase;
    font-weight: 400;
}

.page .heading span {
    display: inline-block;
    animation: showSpans 1.5s ease forwards;
}

.page .heading span:nth-of-type(1) {
    transform: translateX(-200%);
    animation-delay: 2s;
}

.page .heading span:nth-of-type(2) {
    font-weight: bold;
    transform: translateY(-200%);
    animation-delay: 3s;
}

.page .heading span:nth-of-type(3) {
    transform: translateY(200%);
    animation-delay: 3s;
}

.page .scroll {
    position: absolute;
    top: 70%;
    left: 10%;
}

.page .scroll small {
    writing-mode: vertical-lr;
    font-weight: 700;
    letter-spacing: 3px;
    color: rgb(102, 102, 96);
}

.page .photo-container {
    position: fixed;
    top: 55%;
    left: 50%;
    transform: var(--fix-topLeft) rotateZ(38deg);
    transition: all 1.5s ease;
    width: 230px;
    height: 300px;
    background-color: #fff;
    box-shadow: 1px 1px 6px rgba(36, 35, 35, 0.1),
                -1px -1px 6px rgba(206, 206, 206, 0.1);
    animation: photoThrow 2.8s ease;
    &.right {
        top: 47%;
        left: 75%;
        transition: all 1.5s ease;
    }
}

.page .photo-container .photo {
    position: relative;
    margin: 20px auto;
    width: 80%;
    height: 75%;
    opacity: 0.6;
}

.page .photo-container img {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 100;
}

.page .photo-container .after {
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: rgba(255, 255, 255, 0.2);
    z-index: 500;
}

.page .photo-container .hide-eyes {
    position: absolute;
    top: 43%;
    left: 32%;
    width: 65px;
    height: 15px;
    background-color: rgb(20, 20, 20);
    z-index: 500;
}

.page .photo-container .name {
    position: absolute;
    bottom: 5%;
    left: 50%;
    transform: translateX(-50%);
    opacity: 0.3;
    font-family: var(--f1-family);
    letter-spacing: 2px;
    z-index: 500;
}

.page .info-about {
    line-height: 1.75em;
    letter-spacing: 0.3px;
    color: rgb(99, 99, 99);
}

.page .hello {
    position: absolute;
    left: 15%;
    max-height: 80vh;
    text-align: left;
    font-size: 2em;
    letter-spacing: 5px;
    padding: 2rem;
    text-transform: uppercase;
    & h1 span {
        display: inline-block;
        transform: translateY(100px);
        opacity: 0;
        transition: 1s ease;
    }
    &.show h1 span {
        transform: translateY(0);
        opacity: 1;
        transition: 1s ease;
    }
}

/* .page .hello h1 {
    margin: 0 2rem;
} */

.page .hello h1 span {
    display: inline-block;
    transform: translateY(100px);
    opacity: 0;
    transition: 1s ease;
}

.page .hello.show h1 span {
    transform: translateY(0);
    opacity: 1;
    transition: 1s ease;
}

.page .text {
    display: flex;
    align-items: flex-start;
    /* margin-top: 0rem; */
}

.page .text p {
    font-size: 14px;
    margin: 0.7rem 0;
    width: 35vw;
    line-height: 1.5rem;
    letter-spacing: 0.3px;
    color: rgba(99, 99, 99, 0.85);
    text-transform: none;
    text-align: left;
}

.bars-menu {
    position: fixed;
    top: 0;
    height: 100vh;
    width: 100vw;
    display: none;
    z-index: 100;
    background-color: #fff0fe;
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
    color: var(--sec-color-dark);
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
    color: var(--sec-color-dark);
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
    color: rgba(0, 0, 0, 0.6);
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
    background-color: var(--sec-color-dark);
}

.cursor--canvas {
    width: 100vw;
    height: 100vh;
    z-index: 12000;
}

body, body a, body button {
    cursor: none;
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

@keyframes hideStyle {
    /* 0%   {background-color: #fff;}
    10%  {background-color: #000;}
    20%  {background-color: #fff;}
    30%  {background-color: #000;}
    40%  {background-color: #fff;}
    50%  {background-color: #000;}
    60%  {opacity: 0;}
    100% {opacity: 1;} */
    from {
        background-color: blue;
    }
    to {
        background-color: red;
    }
}

@keyframes photoThrow {
    from {
        top: -50%;
        transform: var(--fix-topLeft) rotateZ(0deg);       
    }
}

@keyframes showSpans {
    to {
        transform: translate(0, 0);
    }
}

@media(max-width: 800px) {
    .page .heading {
        transform: var(--fix-topLeft) scale(0.8);
    }

    .page .text {
        flex-direction: column;
    }

    .page .text p {
        width: 80vw;
        margin: 0.5em;
    }
}

@media(max-width: 565px) {
    .hom, .nav.bottom, .nav.top a {
        display: none;
    }

    .nav .home a {
        display: block;
    }

    .cursor--small {
        opacity: 0;
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
        transform: translateX(-100vw);
        display: block;
    }

    .bars-menu.show {
        transform: translateX(0);
    }

    .bars-menu .link-m a {
        opacity: 0;
        transform: translateX(-10vw);
        transition: 1s ease;
    }

    .bars-menu.show .link-m a {
        opacity: 1;
        transform: translateX(0);
        transition: transform 1s ease,
                    opacity 1s ease;
    }

    .bars-menu.show .link-m a:nth-of-type(1) {
        transition-delay: 0.4s;
    }

    .bars-menu.show .link-m a:nth-of-type(2) {
        transition-delay: 0.5s;
    }

    .bars-menu .email-m button {
        background-color: transparent;
        border: none;
        margin-top: 2em;
        font-size: 1em;
        color: var(--sec-color-dark);
    }

    .bars-menu .email-m a {
        font-size: 18px;
        text-transform: none;
    }

    .nav a:hover {
        font-weight: 400;
    }

    .page .heading {
        transform: var(--fix-topLeft) scale(0.4);
    }

    .info-about {
        width: 90%;
    }

    .page .text {
        flex-direction: column;
        top: 60%;
    }
    
    .page .photo-container {
        top: 50%;
        transform: var(--fix-topLeft) rotateZ(38deg) scale(0.8);
    }

    .page .photo-container.right {
        top: 50%;
        left: 50%;
    }

    .page .hello {
        font-size: 1.3em;
        text-align: center;
        left: 0;
    }

    .page .text p {
        font-size: 14px;
        width: 80vw;
        margin: 0.5em;
        line-height: 1.75em;
    }
}

@media(max-width: 380px) {
    .link-m {
        transform: scale(0.8);
        left: -2%;
        top: 78%;
    }

    .email-m {
        transform: translate(-50%, -50%) scale(-1, -1) scale(0.8);
    }

    .page .hello {
        transform: scale(0.9);
    }

    .page .photo-container {
        transform: var(--fix-topLeft) rotateZ(38deg) scale(0.7);
    }

    .page .text p {
        width: 100%;
    }
}

@media(max-width: 300px) {
    .page .heading {
        transform: var(--fix-topLeft) scale(0.3);
    }
}
 </style>