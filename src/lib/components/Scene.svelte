<script>
  import { T, useTask, useLoader } from '@threlte/core'
  import { OrbitControls } from '@threlte/extras'
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
  import {Mesh, ShaderMaterial} from 'three'

  const gltf = useLoader(GLTFLoader).load('/models/brain.glb')
</script>

<T.PerspectiveCamera
  makeDefault
  position={[6, 6, 10]}
  oncreate={(ref) => {
    ref.lookAt(0, 1, 0);
  }}
>
<OrbitControls
  target={[0, 1, 0]}
  enablePan={false}
  enableZoom={false}
  autoRotate={true}
  minPolarAngle={Math.PI / 3}
  maxPolarAngle={Math.PI / 3}
/>
</T.PerspectiveCamera>

{#if $gltf}
  <T
    is={$gltf.scene}
    scale={10}
    oncreate={(scene) => {
      scene.traverse((child) => {
        // Type guard: only assign material if child is a Mesh
        if (child instanceof Mesh) {
          // Example: simple custom shader material
            child.material = new ShaderMaterial({
              vertexShader: `
                varying vec3 vNormal;
                varying vec3 vPosition;
                void main() {
                  vNormal = normalize(normalMatrix * normal);
                  vPosition = (modelViewMatrix * vec4(position, 1.0)).xyz;
                  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
                }
              `,
              fragmentShader: `
                varying vec3 vNormal;
                varying vec3 vPosition;
                void main() {
                  // Dramatic lighting with strong reflections
                  vec3 lightDir1 = normalize(vec3(0.0, 1.0, 1.0));
                  vec3 lightDir2 = normalize(vec3(-1.0, 0.5, 0.0));
                  vec3 viewDir = normalize(-vPosition);

                  float diff1 = max(dot(normalize(vNormal), lightDir1), 0.0);
                  float diff2 = max(dot(normalize(vNormal), lightDir2), 0.0);

                  // Strong, colored specular highlights for dramatic effect
                  vec3 reflectDir1 = reflect(-lightDir1, normalize(vNormal));
                  vec3 reflectDir2 = reflect(-lightDir2, normalize(vNormal));
                  float spec1 = pow(max(dot(viewDir, reflectDir1), 0.0), 64.0);
                  float spec2 = pow(max(dot(viewDir, reflectDir2), 0.0), 32.0);

                  // Keep base color black
                  vec3 baseColor = vec3(0.0, 0.0, 0.0);

                    // Dramatic white and darker purple specular highlights
                    vec3 specular = vec3(1.0, 1.0, 1.0) * spec1 * 2.0 + vec3(0.4, 0.1, 0.5) * spec2 * 1.2;

                  // Strong ambient occlusion for depth
                  float occlusion = 0.5 + 0.5 * vNormal.y;

                  // Rim light for dramatic edge
                  float rim = 1.0 - max(dot(viewDir, normalize(vNormal)), 0.0);
                  rim = pow(rim, 2.5);
                  vec3 rimColor = vec3(1.0, 0.7, 1.0) * rim * 1.2;

                  // Combine: black base, dramatic specular, rim, and visible with light
                  vec3 color = baseColor + specular + rimColor;
                  color *= occlusion;

                  // Ensure minimum visibility in dark areas
                  color = max(color, vec3(0.08, 0.08, 0.08));

                  gl_FragColor = vec4(color, 1.0);
                }
              `
            });
        }
      });
    }}
  />
{/if}


