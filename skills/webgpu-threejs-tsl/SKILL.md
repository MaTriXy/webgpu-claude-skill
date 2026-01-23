---
name: webgpu-threejs-tsl
description: Comprehensive guide for developing WebGPU-enabled Three.js applications using TSL (Three.js Shading Language). Covers WebGPU renderer setup, TSL syntax and node materials, compute shaders, post-processing effects, and WGSL integration. Use this skill when working with Three.js WebGPU, TSL shaders, node materials, or GPU compute in Three.js.
---

# WebGPU Three.js with TSL (Three.js Shading Language)

This skill provides comprehensive knowledge for developing WebGPU-enabled applications using Three.js and TSL (Three.js Shading Language).

## Overview

TSL is a node-based shader abstraction written in JavaScript that simplifies GPU shader creation compared to traditional GLSL. It allows developers to write shader logic directly in JavaScript without string manipulation, benefiting from type checking and automatic optimization.

### Key Advantages

- **JavaScript Integration**: Write shader logic in JS/TS without string manipulation
- **Automatic Optimization**: Type conversions, temporary variables, varying declarations handled automatically
- **Cross-Backend**: Compiles to both WGSL (WebGPU) and GLSL (WebGL)
- **Composable**: Functions can be chained and combined like functional programming

## Project Setup

### Installation

```bash
npm install three
```

### WebGPU Imports

```javascript
// WebGPU-specific Three.js entry point
import * as THREE from 'three/webgpu';

// TSL functions
import {
  // Types and constructors
  float, int, uint, bool, vec2, vec3, vec4, color, uniform,
  mat2, mat3, mat4,

  // Geometry attributes
  positionLocal, positionWorld, positionView, positionGeometry,
  normalLocal, normalWorld, normalView, normalGeometry,
  tangentLocal, tangentWorld, tangentView,
  uv,

  // Camera
  cameraPosition, cameraNear, cameraFar, cameraViewMatrix, cameraProjectionMatrix,

  // Screen
  screenUV, screenCoordinate, screenSize,

  // Time
  time, deltaTime,

  // Math functions
  abs, sin, cos, tan, asin, acos, atan, sqrt, pow, exp, log,
  min, max, clamp, mix, step, smoothstep,
  normalize, length, distance, dot, cross, reflect, refract,
  floor, ceil, fract, mod, sign,

  // Texture
  texture, textureLoad, textureSize, cubeTexture, texture3D,

  // Control flow
  Fn, If, Loop, Break, Continue, Return, Discard,
  select,

  // Oscillators
  oscSine, oscSquare, oscTriangle, oscSawtooth,

  // Effects
  grayscale, saturation, hue, vibrance, luminance, posterize,

  // Compute
  instancedArray, instanceIndex, workgroupBarrier, storageBarrier,
  atomicAdd, atomicSub, atomicMax, atomicMin,

  // Utilities
  hash, checker, remap, range, rotate,

  // Post-processing
  pass, output,

  // WGSL interop
  wgslFn
} from 'three/tsl';

// Post-processing effects from addons
import { bloom } from 'three/addons/tsl/display/BloomNode.js';
import { gaussianBlur } from 'three/addons/tsl/display/GaussianBlurNode.js';
```

### Basic WebGPU Setup

```javascript
import * as THREE from 'three/webgpu';

async function init() {
  // Create WebGPU renderer
  const renderer = new THREE.WebGPURenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
  document.body.appendChild(renderer.domElement);

  // Initialize WebGPU (required!)
  await renderer.init();

  // Create scene and camera
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  camera.position.z = 5;

  // Animation loop
  function animate() {
    renderer.render(scene, camera);
  }
  renderer.setAnimationLoop(animate);
}

init();
```

## TSL Core Concepts

### Types and Constructors

```javascript
// Scalar types
const f = float(1.0);
const i = int(42);
const u = uint(100);
const b = bool(true);

// Vector types
const v2 = vec2(1.0, 2.0);
const v3 = vec3(1.0, 2.0, 3.0);
const v4 = vec4(1.0, 2.0, 3.0, 1.0);

// Color (RGB)
const c = color(0xff0000);  // or color(1, 0, 0)

// Matrix types
const m3 = mat3();
const m4 = mat4();

// Type conversion
const converted = v3.toVec4(1.0);
const asFloat = i.toFloat();
```

### Uniforms

```javascript
// Create uniforms with different types
const myColor = uniform(new THREE.Color(0x0066ff));
const myFloat = uniform(0.5);
const myVec3 = uniform(new THREE.Vector3(1, 2, 3));

// Update uniforms at runtime
myColor.value.set(0xff0000);
myFloat.value = 0.8;

// Update events
const autoUpdated = uniform(0).onFrameUpdate((frame) => Math.sin(frame.time));
const perObject = uniform(0).onObjectUpdate((object) => object.userData.value);
```

### Vector Swizzling

```javascript
const v = vec3(1.0, 2.0, 3.0);

// Access components
const x = v.x;        // 1.0
const xy = v.xy;      // vec2(1.0, 2.0)
const rgb = v.rgb;    // same as xyz

// Reorder components
const zyx = v.zyx;    // vec3(3.0, 2.0, 1.0)
const xxy = v.xxy;    // vec3(1.0, 1.0, 2.0)
```

### Operators

```javascript
// Arithmetic (method chaining)
const result = a.add(b).mul(c).sub(d).div(e);

// Comparison
const isGreater = a.greaterThan(b);
const isLess = a.lessThan(b);
const isEqual = a.equal(b);

// Assignment (for variables)
const v = vec3(0).toVar();
v.assign(vec3(1, 2, 3));
v.addAssign(vec3(1));
v.mulAssign(2.0);

// Logical
const combined = a.and(b).or(c);
const negated = a.not();
```

## Node Materials

### Basic Usage

```javascript
// Create node material
const material = new THREE.MeshStandardNodeMaterial();

// Set color from texture
material.colorNode = texture(colorMap);

// Set color from position (gradient effect)
material.colorNode = positionLocal.normalize();

// Animated color
material.colorNode = vec3(
  oscSine(time),
  oscSine(time.add(0.33)),
  oscSine(time.add(0.66))
);
```

### Material Properties

```javascript
const material = new THREE.MeshStandardNodeMaterial();

// Basic properties
material.colorNode = texture(diffuseMap);
material.opacityNode = float(0.8);
material.alphaTestNode = float(0.5);
material.normalNode = normalMap(texture(normalMapTex));

// PBR properties
material.metalnessNode = texture(metalMap).r;
material.roughnessNode = texture(roughMap).r;
material.emissiveNode = color(0xff0000).mul(emissiveStrength);

// Advanced (MeshPhysicalNodeMaterial)
const physicalMat = new THREE.MeshPhysicalNodeMaterial();
physicalMat.clearcoatNode = float(1.0);
physicalMat.transmissionNode = float(0.9);
physicalMat.iridescenceNode = float(1.0);
physicalMat.sheenNode = vec3(1.0, 0.5, 0.5);

// Displacement
material.positionNode = positionLocal.add(
  normalLocal.mul(texture(displacementMap).r.mul(0.1))
);
```

### Available Node Materials

- `MeshBasicNodeMaterial` - Unlit material
- `MeshStandardNodeMaterial` - PBR material
- `MeshPhysicalNodeMaterial` - Advanced PBR with clearcoat, transmission, etc.
- `MeshPhongNodeMaterial` - Phong shading
- `MeshToonNodeMaterial` - Toon/cel shading
- `PointsNodeMaterial` - For point clouds
- `LineBasicNodeMaterial` - For lines
- `SpriteNodeMaterial` - For sprites

## Custom Functions with Fn()

### Basic Functions

```javascript
// Simple function
const myFunction = Fn(([a, b]) => {
  return a.add(b).mul(2.0);
});

// Use the function
material.colorNode = myFunction(vec3(1, 0, 0), vec3(0, 1, 0));
```

### Functions with Defaults

```javascript
const oscillate = Fn(([frequency = 1.0, amplitude = 1.0]) => {
  return time.mul(frequency).sin().mul(amplitude);
});

// Call with defaults
const result1 = oscillate();
const result2 = oscillate(2.0);
const result3 = oscillate(2.0, 0.5);
```

### Complex Shader Functions

```javascript
// Fresnel effect
const fresnel = Fn(([power = 2.0]) => {
  const viewDir = cameraPosition.sub(positionWorld).normalize();
  const dotNV = normalWorld.dot(viewDir).saturate();
  return float(1.0).sub(dotNV).pow(power);
});

// Apply rim lighting
const rimLight = fresnel(3.0).mul(color(0x00ffff));
material.emissiveNode = rimLight;
```

## Control Flow

### Conditionals

```javascript
// If-Else
const result = vec3(0).toVar();

If(value.greaterThan(0.5), () => {
  result.assign(vec3(1, 0, 0));
}).ElseIf(value.greaterThan(0.25), () => {
  result.assign(vec3(0, 1, 0));
}).Else(() => {
  result.assign(vec3(0, 0, 1));
});

// Ternary (select)
const color = select(condition, trueValue, falseValue);
```

### Loops

```javascript
// Simple loop
const sum = float(0).toVar();
Loop(10, ({ i }) => {
  sum.addAssign(float(i).div(10));
});

// Ranged loop
Loop({ start: int(0), end: int(count), type: 'int' }, ({ i }) => {
  // Loop body
});

// Nested loops
Loop(width, height, ({ i, j }) => {
  // Access i for outer, j for inner
});

// Loop control
Loop(100, ({ i }) => {
  If(condition, () => {
    Break();
  });
  If(skipCondition, () => {
    Continue();
  });
});
```

## Texturing

### Basic Texture Sampling

```javascript
// Sample texture at UV
material.colorNode = texture(diffuseMap, uv());

// Animated UVs
const animatedUV = uv().add(vec2(time.mul(0.1), 0));
material.colorNode = texture(diffuseMap, animatedUV);

// Scale UVs
const scaledUV = uv().mul(4.0);  // Tile 4x4
material.colorNode = texture(diffuseMap, scaledUV);
```

### Advanced Texturing

```javascript
// Triplanar mapping (for terrain/rocks)
material.colorNode = triplanarTexture(
  texture(diffuseMap),
  null,  // optional Y-axis texture
  null,  // optional Z-axis texture
  float(0.1)  // sharpness
);

// Cube map
material.envNode = cubeTexture(envMap);

// Screen-space projection
material.colorNode = texture(diffuseMap, screenUV);
```

## Compute Shaders

### Setup

```javascript
// Create storage buffers
const count = 100000;
const positions = instancedArray(count, 'vec3');
const velocities = instancedArray(count, 'vec3');
```

### Initialize Compute Shader

```javascript
const computeInit = Fn(() => {
  const position = positions.element(instanceIndex);
  const velocity = velocities.element(instanceIndex);

  // Initialize with random positions
  position.x.assign(hash(instanceIndex).mul(10).sub(5));
  position.y.assign(hash(instanceIndex.add(1)).mul(10).sub(5));
  position.z.assign(hash(instanceIndex.add(2)).mul(10).sub(5));

  // Zero velocity
  velocity.assign(vec3(0));
})().compute(count);

// Run once at startup
await renderer.computeAsync(computeInit);
```

### Update Compute Shader

```javascript
const gravity = uniform(-9.8);
const deltaTimeUniform = uniform(0);

const computeUpdate = Fn(() => {
  const position = positions.element(instanceIndex);
  const velocity = velocities.element(instanceIndex);

  // Apply gravity
  velocity.y.addAssign(gravity.mul(deltaTimeUniform));

  // Update position
  position.addAssign(velocity.mul(deltaTimeUniform));

  // Ground collision
  If(position.y.lessThan(0), () => {
    position.y.assign(0);
    velocity.y.assign(velocity.y.negate().mul(0.8));  // Bounce
  });
})().compute(count);

// In animation loop
function animate() {
  deltaTimeUniform.value = clock.getDelta();
  renderer.compute(computeUpdate);
  renderer.render(scene, camera);
}
```

### Using Compute Results in Materials

```javascript
// Create instanced mesh using computed positions
const geometry = new THREE.SphereGeometry(0.1);
const material = new THREE.MeshStandardNodeMaterial();

// Read position from compute buffer
material.positionNode = positions.element(instanceIndex);

const mesh = new THREE.InstancedMesh(geometry, material, count);
scene.add(mesh);
```

## Post-Processing

### Basic Setup

```javascript
import * as THREE from 'three/webgpu';
import { pass } from 'three/tsl';

// Create post-processing
const postProcessing = new THREE.PostProcessing(renderer);

// Create scene pass
const scenePass = pass(scene, camera);
const scenePassColor = scenePass.getTextureNode('output');

// Simple passthrough
postProcessing.outputNode = scenePassColor;
```

### Built-in Effects

```javascript
import { bloom } from 'three/addons/tsl/display/BloomNode.js';
import { gaussianBlur } from 'three/addons/tsl/display/GaussianBlurNode.js';
import { fxaa } from 'three/addons/tsl/display/FXAANode.js';

// Bloom
const bloomPass = bloom(scenePassColor);
bloomPass.threshold.value = 0.5;
bloomPass.strength.value = 1.0;
bloomPass.radius.value = 0.5;

// Combine original + bloom
postProcessing.outputNode = scenePassColor.add(bloomPass);

// Blur
const blurred = gaussianBlur(scenePassColor, vec2(2.0));

// Grayscale
const gray = grayscale(scenePassColor);

// Chain effects
postProcessing.outputNode = fxaa(scenePassColor.add(bloomPass));
```

### Custom Post-Processing

```javascript
const customEffect = Fn(() => {
  const color = scenePassColor.toVar();

  // Vignette
  const uv = screenUV;
  const dist = uv.sub(0.5).length();
  const vignette = float(1.0).sub(dist.mul(0.5));
  color.rgb.mulAssign(vignette);

  // Color grading
  color.r.mulAssign(1.1);  // Slight red boost

  return color;
});

postProcessing.outputNode = customEffect();
```

### Render to Multiple Targets (MRT)

```javascript
import { mrt } from 'three/tsl';

const scenePass = pass(scene, camera);

// Set up MRT
scenePass.setMRT(mrt({
  output: output,
  normal: normalView
}));

// Access individual targets
const colorTexture = scenePass.getTextureNode('output');
const normalTexture = scenePass.getTextureNode('normal');
```

## WGSL Integration

### Custom WGSL Functions

```javascript
import { wgslFn } from 'three/tsl';

// Define WGSL function
const customWGSL = wgslFn(`
  fn customFunction(color: vec3<f32>, factor: f32) -> vec3<f32> {
    let adjusted = pow(color, vec3<f32>(1.0 / factor));
    return clamp(adjusted, vec3<f32>(0.0), vec3<f32>(1.0));
  }
`);

// Use in TSL
material.colorNode = customWGSL(texture(diffuseMap).rgb, float(2.2));
```

## Common Patterns

### Animated Gradient

```javascript
material.colorNode = mix(
  color(0x0000ff),
  color(0xff0000),
  positionLocal.y.add(time.mul(0.5).sin()).mul(0.5).add(0.5)
);
```

### Procedural Noise Pattern

```javascript
const noise = Fn(([p]) => {
  return fract(p.dot(vec3(12.9898, 78.233, 45.543)).sin().mul(43758.5453));
});

material.colorNode = vec3(noise(positionWorld.mul(10)));
```

### Fresnel Rim Lighting

```javascript
const viewDir = cameraPosition.sub(positionWorld).normalize();
const fresnel = float(1.0).sub(normalWorld.dot(viewDir).saturate()).pow(3.0);
material.emissiveNode = color(0x00ffff).mul(fresnel);
```

### UV Distortion

```javascript
const distortedUV = uv().add(
  vec2(
    time.mul(0.1).sin().mul(0.1),
    time.mul(0.15).cos().mul(0.1)
  )
);
material.colorNode = texture(diffuseMap, distortedUV);
```

### Dissolve Effect

```javascript
const noiseValue = hash(positionLocal.mul(50));
const threshold = uniform(0.5);

If(noiseValue.lessThan(threshold), () => {
  Discard();
});

// Edge glow
const edgeWidth = 0.1;
const edge = smoothstep(threshold, threshold.add(edgeWidth), noiseValue);
material.emissiveNode = color(0xff5500).mul(float(1.0).sub(edge));
```

## Geometry Attributes Reference

### Position

| Node | Description |
|------|-------------|
| `positionGeometry` | Original vertex position |
| `positionLocal` | Position in local/model space |
| `positionWorld` | Position in world space |
| `positionView` | Position in view/camera space |

### Normal

| Node | Description |
|------|-------------|
| `normalGeometry` | Original vertex normal |
| `normalLocal` | Normal in local space |
| `normalWorld` | Normal in world space |
| `normalView` | Normal in view space |

### Other Attributes

| Node | Description |
|------|-------------|
| `uv()` | Primary UV coordinates |
| `uv(1)` | Secondary UV set |
| `tangentLocal/World/View` | Tangent vectors |
| `bitangentLocal/World/View` | Bitangent vectors |
| `vertexColor()` | Vertex colors |
| `instanceIndex` | Current instance index |
| `vertexIndex` | Current vertex index |

## Camera & Screen Reference

| Node | Description |
|------|-------------|
| `cameraPosition` | Camera position in world space |
| `cameraNear` | Camera near plane |
| `cameraFar` | Camera far plane |
| `cameraViewMatrix` | Camera view matrix |
| `cameraProjectionMatrix` | Camera projection matrix |
| `screenUV` | Screen-space UV (0-1) |
| `screenCoordinate` | Screen-space pixel coordinates |
| `screenSize` | Screen dimensions |

## Math Functions Reference

### Basic Math

`abs`, `sign`, `floor`, `ceil`, `fract`, `mod`, `min`, `max`, `clamp`

### Trigonometry

`sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`

### Exponential

`pow`, `exp`, `log`, `sqrt`, `inverseSqrt`

### Interpolation

`mix(a, b, t)`, `step(edge, x)`, `smoothstep(edge0, edge1, x)`

### Vector Math

`length`, `distance`, `dot`, `cross`, `normalize`, `reflect`, `refract`, `faceforward`

### Constants

`PI`, `TWO_PI`, `HALF_PI`, `EPSILON`

## Example: Complete Earth Shader

```javascript
import * as THREE from 'three/webgpu';
import {
  texture, uv, normalWorld, positionWorld, cameraPosition,
  color, uniform, mix, step, smoothstep, pow, normalize, dot,
  bumpMap, float
} from 'three/tsl';

// Load textures
const loader = new THREE.TextureLoader();
const dayTexture = loader.load('earth_day.jpg');
const nightTexture = loader.load('earth_night.jpg');
const cloudsTexture = loader.load('earth_clouds.jpg');
const bumpTexture = loader.load('earth_bump.jpg');

// Uniforms
const sunDirection = uniform(new THREE.Vector3(1, 0, 0));
const atmosphereColor = uniform(color(0x4db2ff));

// Create material
const earthMaterial = new THREE.MeshStandardNodeMaterial();

// Day/night blending based on sun direction
const sunDot = normalWorld.dot(sunDirection).mul(0.5).add(0.5);
const dayNightMix = smoothstep(0.4, 0.6, sunDot);

const dayColor = texture(dayTexture, uv());
const nightColor = texture(nightTexture, uv());
earthMaterial.colorNode = mix(nightColor, dayColor, dayNightMix);

// Atmosphere fresnel
const viewDir = normalize(cameraPosition.sub(positionWorld));
const fresnel = pow(float(1.0).sub(normalWorld.dot(viewDir).saturate()), 3.0);
earthMaterial.emissiveNode = atmosphereColor.mul(fresnel).mul(0.5);

// Bump mapping
earthMaterial.normalNode = bumpMap(texture(bumpTexture, uv()), 0.05);

// Cloud layer (separate sphere slightly larger)
const cloudMaterial = new THREE.MeshBasicNodeMaterial();
cloudMaterial.colorNode = texture(cloudsTexture, uv());
cloudMaterial.opacityNode = texture(cloudsTexture, uv()).r;
cloudMaterial.transparent = true;
```

## Resources

- [Three.js Documentation](https://threejs.org/docs/)
- [Three.js Shading Language Wiki](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language)
- [Three.js WebGPU Examples](https://github.com/mrdoob/three.js/tree/master/examples) (files prefixed with `webgpu_`)
- [WebGPU Specification](https://www.w3.org/TR/webgpu/)

## License

Three.js is released under the [MIT License](https://github.com/mrdoob/three.js/blob/dev/LICENSE).

Code examples in this skill are derived from the Three.js repository and are subject to the same MIT License.
