---
title: Implement Universe by Three.js
---

# Implement Universe by Three.js

[TOC]

## How does the universe look like?

![zilliverse](https://user-images.githubusercontent.com/83751452/127113680-d5f77906-68f3-40f4-ac67-994234fb9c03.gif)

Here's the link: https://zilliz.com/universe

We want to build a hero universe. This universe should have different stars as heroes. After specifiec duration the camera will randomly focus on one star and the info will appear. Each group of stars revolve around a "Zilliz" star and will appear or disappear periodically, just like blink.

## How to implement universe?

Three.js Docs: https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene

### Brief introduction

Creating the scene => Rendering the scene => Animating the elements

```javascript
export default function renderUniverse() {
  // Define variables & scene here.
  init();
  animate();
  
  function init() {
    // Init scene (including elements) here.
  }
  
  function animate() {
    requestAnimationFrame(animate);
    render();
  }
  
  function render() {
    // Define how elements change in scene.
  }
}
```


### Creating the scene

> To actually be able to display anything with three.js, we need three things: scene, camera and renderer, so that we can render the scene with camera.

```javascript
// functuin init() {
// ...

let camera, scene, renderer;

// Init scene.
scene = new THREE.Scene();

// Init camera.
camera = new THREE.PerspectiveCamera(
  25,
  SCREEN_WIDTH / SCREEN_HEIGHT,
  50,
  1e7
);

// Init renderer.
renderer = new THREE.WebGLRenderer({ antialias: true });

// Then mount the renderer.
document.body.appendChild(renderer.domElement);
```

#### Add elements to this scene

1. Scene Background: `scene.background = new THREE.Color(0x010e29);`
2. Scene Fog: `scene.fog = new THREE.FogExp2(0x000000, 0.00000025);`
3. Direct Light: 

```javascript
const dirLight = new THREE.DirectionalLight(0xffffff);
dirLight.position.set(1, 0, 2).normalize();
scene.add(dirLight);
```

4. Geometry: (There're different geometries and we use `SphereGeometry` here):

```javascript
const material = new THREE.MeshPhongMaterial({
  specular: 0x333333,
  shininess: 15,
  map: textureLoader.load('/images/starLogo.svg'),
  normalScale: new THREE.Vector2(0.85, -0.85),
  transparent: true,
});
const geometry = new THREE.SphereGeometry(
  radius,
  32,
  32
);
const meshPlanet = new THREE.Mesh(geometry, material);
scene.add(meshPlanet);
```

#### Tandem addition

As the code above, we define a scene, and add background, fog, light and geometry to it. But universe has galaxy such as solar system, a star has many planets revolving around it, a planet has many moons... So we can add a `meshPlanet` to a `meshPlanet`, just like tandem addition.

```javascript
// functuin init() {
// ...

const earthMaterial = new THREE.MeshPhongMaterial({
  // emissive: 0x7dcef4,
  map: textureLoader.load(
    `textures/planets/earth.jpg`
  ),
  transparent: true,
  opacity: 1,
});
const meshMoon = new THREE.Mesh(geometry, earthMaterial);
const rand = Math.random();
const moonRadian = Math.PI * 2 * rand;
meshMoon.position.set(
  moonRadius * Math.sin(moonRadian),
  0,
  moonRadius * Math.cos(moonRadian)
);
meshMoon.scale.set(
  moonScale * rand,
  moonScale * rand,
  moonScale * rand
);
meshMoon.resetScale = function () {
  const size = moonScale * rand * 2;
  this.scale.set(size, size, size);
};
meshPlanet.add(meshMoon);
```

Then we have a meshPlanet with a meshMoon, we can add more different meshMoon here and it will look more real.

After tandem addition, if you change the meshPlanet's position or rotate it, all children will change with it, so that we can make a moving galaxy.

### Customize Scene

#### Define different tracks

If we have three transparent planets in the same position, each planet has a number of moons with different tilt angles, directions and radius, it will look more vivid.

```javascript
// Here's the example of defination, the changes need to be set up in render().
// functuin init() {
// ...

const meshPlanets = [];
for (let i = 0; i < 3; i++) {
  const emptyMaterialNormalMap = new THREE.MeshPhongMaterial({
    transparent: true,
    opacity: 0,
  });
  const tempMeshPlanet = new THREE.Mesh(
    meshPlanetGeometry,
    emptyMaterialNormalMap
  );
  meshPlanets.push(tempMeshPlanet);
  scene.add(tempMeshPlanet);
}
meshPlanets.forEach(planet => {
  const tempMoonsCollection = [];
  for (var i = 0; i < 4; i++) {
    const rand = Math.random();
    const rand1 = Math.random();
    const moonRadius = radius * 5 * rand;
    const moonRadian = Math.PI * 2 * rand1;

    const randomPlanetMaterial = new THREE.MeshPhongMaterial({
      emissive: 0x7dcef4,
      transparent: true,
      opacity: 1,
    });

    meshMoon1 = new THREE.Mesh(geometry, randomPlanetMaterial);
    meshMoon1.position.set(
      moonRadius * Math.sin(moonRadian),
      0,
      moonRadius * Math.cos(moonRadian)
    );
    meshMoon1.scale.set(
      moonScale * rand,
      moonScale * rand,
      moonScale * rand
    );
    meshMoon1.resetScale = function () {
      const size = moonScale * rand * 2;
      this.scale.set(size, size, size);
    };
    tempMoonsCollection.push(meshMoon1);
    moons.push(meshMoon1);
  }
  moonsCollection.push(tempMoonsCollection);

  planet.rotation.x -= Math.random();
  planet.rotation.z += 0.3;
});
```

#### Define background with little stars

```javascript
// functuin init() {
// ...

const starsGeometry = new THREE.BufferGeometry();
const vertex = new THREE.Vector3();
const vertices = [];

for (let i = 0; i < 250; i++) {
  vertex.x = Math.random() * 2 - 1;
  vertex.y = Math.random() * 2 - 1;
  vertex.z = Math.random() * 2 - 1;
  vertex.multiplyScalar(r);

  vertices.push(vertex.x, vertex.y, vertex.z);
}

starsGeometry.setAttribute(
  'position',
  new THREE.Float32BufferAttribute(vertices, 3)
);

const starsMaterials = new THREE.PointsMaterial({
  color: 0x555555,
  size: 2,
  sizeAttenuation: false,
});

for (let i = 10; i < 300; i++) {
  const stars = new THREE.Points(
    starsGeometry,
    starsMaterials
  );

  stars.rotation.x = Math.random() * 6;
  stars.rotation.y = Math.random() * 6;
  stars.rotation.z = Math.random() * 6;
  stars.scale.setScalar(i * 3);

  stars.matrixAutoUpdate = false;
  stars.updateMatrix();

  scene.add(stars);
}
```

### Customize rendering

#### Make moons rotating

```javascript
const clock = new THREE.Clock();

// function render() {
// ...

const rotationSpeed = 0.025;
const delta = clock.getDelta();

meshPlanet.rotation.y -= rotationSpeed * 15 * delta;

meshPlanets.forEach(
  (planet, index) =>
    (planet.rotation.y -= rotationSpeed * (index * 2 + 3) * 0.25 * delta)
);
```

#### Make moons blinking

Randomly choose a set of moons(added to the same planet before)

```javascript
const STATIC_BLINK_FRAMES = 128;
let blinkFrames = STATIC_BLINK_FRAMES;
let shouldAddOpacity = false;
let blinkList = [];

// function render() {
// ...

// Raise opacity and the star will be brighter.
if (shouldAddOpacity) {
  blinkList.forEach(
    targetMoon => (targetMoon.material.opacity += 1 / STATIC_BLINK_FRAMES)
  );
  blinkFrames++;
// Reduce opacity and the star will be darker.
} else {
  blinkList.forEach(
    targetMoon => (targetMoon.material.opacity -= 1 / STATIC_BLINK_FRAMES)
  );
  blinkFrames--;
}
if (blinkFrames <= -STATIC_BLINK_FRAMES) {
  shouldAddOpacity = true;
} else if (blinkFrames > STATIC_BLINK_FRAMES * 2) {
  shouldAddOpacity = false;
  blinkList.forEach(targetMoon => (targetMoon.material.opacity = 1));
  blinkFrames = STATIC_BLINK_FRAMES;
  blinkList = [];
}
```

#### Let camera moves more smoothly

If we need move camera from position A to B, we can divide the process into a number of pieces, and camera only complete one piece movement in one frame. Smaller the piece is, more smoothly.

Besides, we need to change the camera angle to fit the movement, so we can divide the process as same as above.

```javascript
function lookAtObj(camera, object) {
  const boundingBox = new THREE.Box3();
  boundingBox.setFromObject(object);
  const center = boundingBox.getCenter(new THREE.Vector3());
  camera.lookAt(center);
}
```

```javascript
let movingCamera = false;
let camCurrnetStep = 0;
let camSpeed = 128; // One movement process will be divided into 128 pieces.

// function render() {
// ...

function updateCamPosition() {
  if (camCurrnetStep >= camSpeed) {
    movingCamera = false;
    camCurrnetStep = 0;
    return;
  }
  camera.position.set(
    camera.position.x + camStepFov[0],
    camera.position.y + camStepFov[1],
    camera.position.z + camStepFov[2]
  );
  targetObj && lookAtObj(camera, targetObj);
  camCurrnetStep++;
}

movingCamera && updateCamPosition();
```

#### Add glow to target moon

We want to add glow to each star when focused, and the glow should smoothly appear and disappear. So that the user can obviously and easily find the target star.

```javascript
// function init() {
// ...
const glowMaterial = new THREE.ShaderMaterial({
  uniforms: {
    c: { type: 'f', value: 0.2 },
    p: { type: 'f', value: 1.0 },
    glowColor: { type: 'c', value: new THREE.Color(0x06aff2) },
    viewVector: { type: 'v3', value: camera.position },
  },
  // Shader
  vertexShader: `uniform vec3 viewVector;
    uniform float c;
    uniform float p;
    varying float intensity;
    void main() 
    {
      vec3 vNormal = normalize( normalMatrix * normal );
      vec3 vNormel = normalize( normalMatrix * viewVector );
      intensity = pow( c - dot(vNormal, vNormel), p );
      gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );
    }`,
  fragmentShader: `uniform vec3 glowColor;
    varying float intensity;
    void main() 
    {
      vec3 glow = glowColor * intensity;
      gl_FragColor = vec4( glow, 1.0 );
    }`,
  side: THREE.BackSide,
  blending: THREE.AdditiveBlending,
  transparent: true,
});

const moonGlow = new THREE.Mesh(geometry, glowMaterial);
moonGlow.position.set(...);
moonGlow.scale.set(...);
meshMoon.targetGlow = moonGlow; // Add the glow to moon's property here will make it easier for us to change it.
planet.add(meshMoon);
```



```javascript
let shouldAddGlow = false;

// function render() {
// ...

function updateTargetGlow() {
  const glowStep = 0.005;
  if (!targetGlow) return;
  if (shouldAddGlow) {
    targetGlow.material.uniforms.c.value += glowStep;
  } else {
    targetGlow.material.uniforms.c.value -= glowStep;
  }
  if (targetGlow.material.uniforms.c.value >= 0.3) {
    shouldAddGlow = false;
    targetGlow.material.uniforms.c.value = 0.3;
  }
  if (targetGlow.material.uniforms.c.value <= 0.1) {
    shouldAddGlow = true;
    targetGlow.material.uniforms.c.value = 0.1;
  }
  // targetGlow.material.uniforms.c.value = nextUniformsC;
  // targetGlow.updateMatrix();
}

updateTargetGlow();
```

#### Resize automatically

```javascript
window.addEventListener('resize', onWindowResize);

function onWindowResize() {
  SCREEN_WIDTH = document.body.clientWidth - MARGIN * 2;
  if (SCREEN_WIDTH <= 488) {
    SCREEN_HEIGHT = (SCREEN_WIDTH * 4) / 3;
  } else {
    SCREEN_HEIGHT = (SCREEN_WIDTH * 9) / 21;
  }

  camera.aspect = SCREEN_WIDTH / SCREEN_HEIGHT;
  camera.updateProjectionMatrix();

  renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
  composer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
}
```

