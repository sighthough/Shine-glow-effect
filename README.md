# Shine-glow-effect
So the main idea was since we already have shines working on our 3d models why not add a glow effect depending on their intensity etc ? :D







 This project explores how to couple Physically Based Rendering (PBR) specular highlights with a post-processing Unreal Bloom pass to create organic, 
 glowing highlights on 3D surfaces without artificially emitting light from the entire mesh.
 🔬 How It Works (The Technical Concept)Standard WebGL bloom typically applies a glow to the entire scene based on color brightness (RGB > threshold). 
 If an object is bright white, it glows uniformly.
 This engine achieves specular-only blooming by leveraging:
 High Dynamic Range (HDR) Lighting: Light intensities and material specular responses push pixel values well beyond standard LDR range ($> 1.0$).
 ACES Filmic Tone Mapping: Maps HDR values back to displayable range while preserving highlights, allowing peak specular reflections to remain extremely bright.
 Luminosity High-Pass Filtering (UnrealBloomPass): Extracts only those pixels whose luminance exceeds a strict threshold (e.g., 0.4 to 0.8). 
 Because only tight PBR specular pinpricks cross this threshold on smooth metallic surfaces, only the reflections/shines bloom, leaving the diffuse body color intact.
 🛠️ Implementation Guide for DevelopersIf you want to integrate this specular bloom technique into your own Three.js or WebGL project,
 follow this architectural setup:1.
 Configure Renderer with Tone MappingYou must enable tone mapping to handle HDR values. 
 Without ACES Filmic or Reinhardt tone mapping, bright highlights will simply clip to pure white and cause the entire mesh to bloom uniformly.import * as THREE from 'three';

const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

// Crucial for HDR bloom response
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.2;
2. Set Up Post-Processing Effect ComposerStandard renderer.render() must be replaced by an EffectComposer pipeline to inject the bloom shader pass.import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));

// Parameters: (resolution, strength, radius, threshold)
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    1.5,  // Strength
    0.8,  // Radius
    0.4   // Threshold (Controls what luminance triggers the glow)
);
composer.addPass(bloomPass);
3. Material & Light Tuning for Specular BloomTo get the glowing shine effect, use MeshPhysicalMaterial with low roughness, high metalness, and high-intensity directional lights.const material = new THREE.MeshPhysicalMaterial({
    color: 0x0f172a,
    roughness: 0.15,      // Low roughness = sharp, intense microfacet alignment
    metalness: 0.90,      // High metalness = colored specular highlights
    clearcoat: 0.50,      // Adds an extra specular highlight layer
    reflectivity: 1.0,
    specularIntensity: 1.0
});

const geometry = new THREE.TorusKnotGeometry(1.2, 0.38, 128, 32);
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

// High intensity light source to push pixels past the bloom threshold (> 1.0)
const directionalLight = new THREE.DirectionalLight(0x00f0ff, 3.5);
directionalLight.position.set(4, 4, 3);
scene.add(directionalLight);
4. Animation Loop HookIn your render loop, replace renderer.render(scene, camera) with composer.render():function animate() {
    requestAnimationFrame(animate);
    
    // Rotate mesh or animate lights here...
    
    composer.render(); // Renders scene + post-processing bloom pass
}
animate();
⚙️ Key Parameter Tuning GuidelinesParameter
Recommended RangeEffectBloom Threshold0.3 – 0.6Lower values make broader surface areas glow; higher values restrict glow strictly to the sharpest core highlights.
Bloom Strength1.0 – 2.5Controls the intensity of the light scattering halo. Too high will cause artifacting/bleeding.
Material Roughness0.05 – 0.25Lower roughness concentrates reflected light into tight microfacet spikes, triggering intense bloom.
Light Intensity3.0 – 7.0Must be high enough to push surface radiance past the tone mapping and bloom threshold floor.
📦 Project Structure├── index.html       # Complete single-file implementation (HTML, Tailwind CSS, Three.js)
└── 
Open index.html directly in any modern WebGL-enabled browser, or serve it via a local static server (e.g., Live Server in VS Code).
📄 LicenseThis project is open source under the MIT License.
