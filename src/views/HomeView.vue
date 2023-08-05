<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted} from "vue"
import { OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import * as THREE from "three"
import gsap from "gsap"
import * as dat from "dat.gui"

//GUI
//tra le librerie piu popolari ci sono dat.GUI, control-panel, ControlKit, Guify, Oui

//in questo esempio useremo dat.GUI

// Inizializziamo la GUI (Puoi nascondere il pannello premendo H sulla tastiera)
const gui = new dat.GUI()

let canvasRef = ref();

var controls

//Utilizziamo i valori del viewport per impostare la grandezza del canvas
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}

//Aggiungiamo l'event listner per il resize del canvas
window.addEventListener("resize", ()=>{
  //update sizes for the canvas
  sizes.width = window.innerWidth
  sizes.height = window.innerHeight

  //update the camera aspect ratio
  camera.aspect = sizes.width / sizes.height
  camera.updateProjectionMatrix()

  //update renderer
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})


//HANDLE FULL SCREEN (Il codice aggiuntivo è per permettere il funzionamento corretto anche con browser Safari)
window.addEventListener("dblclick",()=>{
  const fullscreenElement = document.fullscreenElement ||  document.webkitFullscreenElement
  if(!fullscreenElement){
    if(canvasRef.value.requestFullscreen){
      canvasRef.value.requestFullscreen()
    }
    else if(canvasRef.value.webkitRequestFullscreen){
      canvasRef.value.webkitRequestFullscreen()
    }
    
  }
  else{
    if(document.exitFullscreen)
    {
      document.exitFullscreen()
    }
    else if(document.webkitExitFullscreen){
      document.webkitExitFullscreen()
    }
  }
})

//RETRIEVE THE MOUSE POSITION
const mouse = new THREE.Vector2()

window.addEventListener("mousemove", (e)=>{
  mouse.x = e.clientX / sizes.width * 2 -1  //use this formula to get a normalized value (that goes from -1 to 1 instead using the pixel value)
  mouse.y = - (e.clientY / sizes.height * 2 - 1)
})

window.addEventListener("click", (e)=>{
  if(currentIntersect){ //in this way an event occur only when you click on something that you can hover

    //and if you want to know on which object you have click just do:
    switch(currentIntersect.object)
    {
      case sphere1:
        console.log("click on sphere1")
        break

      case sphere2:
        console.log("click on sphere2")
        break
        
      case sphere3:
        console.log("click on sphere3")
        break  
    }
  }
})


let scene = new THREE.Scene()

let renderer


//Sphere1
const sphere1 = new THREE.Mesh(
  new THREE.SphereBufferGeometry(0.5,16,16),
  new THREE.MeshBasicMaterial({color:0xff0000})
)
sphere1.position.x -= 2

//Sphere2
const sphere2 = new THREE.Mesh(
  new THREE.SphereBufferGeometry(0.5,16,16),
  new THREE.MeshBasicMaterial({color:0xff0000})
)


//Sphere3
const sphere3 = new THREE.Mesh(
  new THREE.SphereBufferGeometry(0.5,16,16),
  new THREE.MeshBasicMaterial({color:0xff0000})
)
sphere3.position.x += 2

scene.add(sphere1, sphere2, sphere3)



//////////////////// RAYCASTER ////////////////////////
//a raycaster is like a ray in a specific direction, and return what objects intersect with it
//it's used for many things: simulate mouse events, show alert message if you are close to something, test if a laser gun hit something, detect if there is a wall in front of the player (and the distance), and many other

//first of all, create a raycaster
const raycaster = new THREE.Raycaster()

/*
//to shot a ray in a direction, we need to set the origin of the ray, and the direction (in Vector3)
const rayOrigin = new THREE.Vector3(-3, 0, 0)
const rayDirection = new THREE.Vector3(10, 0, 0)
rayDirection.normalize() //used to convert a vector3 , in a normalized vector with the same direction, but lenght 1 (always normalize the direction, the raycaster need that)

//and then provide the raycaster with his property
raycaster.set(rayOrigin, rayDirection)

//cast a ray and see what objects intersect:
//we can use intersectObject (to test only 1 object)
//or intersectObjects (to test what object intersect in an array of objects)

const intersect = raycaster.intersectObject(sphere2)
console.log(intersect)

const intersects = raycaster.intersectObjects([sphere1, sphere2, sphere3])
console.log(intersects)

//the intersectObject / intersectObjects always  return an awway, because a single object can be insersected multiple times (like a torus for example)
//in each item of the list we find this information:
// distance (the distance between the origin of the ray and the collision point)
// point (a vector3 of the exact position of the collision point)
// object (what object is intersecting with the ray)
// uv (the uv coordinates in that geometry, if supported)
// face (what face of the geometry was hit by the ray)
// faceIndex (the index of that face)

//now go in the animate loop to test the raycaster
*/

// CAMERA
let camera = new THREE.PerspectiveCamera(
  75, 
  sizes.width / sizes.height, 
  0.1,
  100  
);
camera.position.set(0,0,3)
scene.add(camera);

const clock = new THREE.Clock()

let currentIntersect = null

//Animate Loop
const animate = () =>{
  const elapsedTime = clock.getElapsedTime()

  sphere1.position.y = Math.sin(elapsedTime * 0.8) * 1.5
  sphere2.position.y = Math.sin(elapsedTime * 0.9) * 1.5
  sphere3.position.y = Math.sin(elapsedTime * 0.7) * 1.5

  /*

  //cast a ray
  const rayOrigin = new THREE.Vector3(-3,0,0)
  const rayDirection = new THREE.Vector3(1,0,0)
  rayDirection.normalize()

  raycaster.set(rayOrigin, rayDirection)

  const objectsToTest = [sphere1, sphere2, sphere3]
  const intersects = raycaster.intersectObjects(objectsToTest)

  //add the logic to rechange the color back to red in each frame in which the sphere doesn't intersect the ray
  for(const object of objectsToTest){
    object.material.color.set("#ff0000")
  }

  //then change the material of the object that insersect with the ray
  for(const intersect of intersects) {
    intersect.object.material.color.set("#0000ff")
  }

  */

  //We can also use the raycaster to test if we are hoovering something with our mouse


  //Hover
  //first we need to get the position of the mouse, we need a value that goes from -1 to +1 in horizontal and vertical axes.
  //so come in the initial section of the script, right below the resize event listener, to initializa a mouse variable
  
  //then we use this method to orientate the raycaster in the right direction , and it's in the right position (the same of the mouse)
  raycaster.setFromCamera(mouse, camera)

  const objectsToTest = [sphere1, sphere2, sphere3]
  const intersects = raycaster.intersectObjects(objectsToTest)

  for(const object of objectsToTest){
    object.material.color.set("#ff0000")
  }

  for(const intersect of intersects){
    intersect.object.material.color.set("#0000ff")
  }
  
  //mouse events like mousenter and mouseleave aren't supported, so we need to simulate it. let's see how we can resolve this problem
  
  //the solution is to create a variavle that hold the currently hovered object

  // if an object intersect, but there wasn't one before, a mouseenter happened
  // if no object intersect, but there was one before, a mouseleave event happened

  //first create this variable (up, just outside the animate loop)
  
  //then return here and 
  if(intersects.length)
  {
    if(currentIntersect === null)
    {
      console.log("mouse enter")
    }
    currentIntersect = intersects[0] //if the ray intersect something, return the first element, in the currentIntersect
  }
  else
  {
    if(currentIntersect){
      console.log("mouse leave")
    }

    currentIntersect = null //else , set che currentIntersect back to null
  }



  //NOW LET'S IMPLEMENT A CLICK EVENT , FOR LISTEN TO THE CLICK EVENT
 //return up , where we instantiate the mouse before


  controls.update()
  
  renderer.render(scene, camera)
}

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });
 
  renderer.setSize(sizes.width, sizes.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio,2)) // in questo modo il max valore di pixel ratio utilizzaro è 2 (altrimenti devi renderizzare troppi pixel, dispiego enorme di potenza gpu)
  renderer.render(scene, camera)
  renderer.setAnimationLoop(animate)

  //CONTROLS
  controls = new OrbitControls(camera, canvasRef.value)
  controls.enableDamping = true; // permette uno effetto smooth una volta che rilasciamo l'elemento, rallenta fino a fermarsi
});

onUnmounted(() => {
  renderer.setAnimationLoop(null)
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>