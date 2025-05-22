<script>
  import { T } from "@threlte/core";
  import { Tween } from "svelte/motion";
  import { quadOut } from "svelte/easing";

  export let node;

  let vertexShader = `
    varying vec3 vNormal;
    varying vec3 vPosition;
    void main() {
      vNormal = normalize(normalMatrix * normal);
      vPosition = (modelViewMatrix * vec4(position, 1.0)).xyz;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `;

  let fragmentShader = `
    varying vec3 vNormal;
    varying vec3 vPosition;
    uniform float hovered;

    void main() {
        vec3 lightDir = normalize(vec3(0.0, 1.0, 1.0));
        vec3 viewDir = normalize(-vPosition);

        float diff = max(dot(vNormal, lightDir), 0.0);
        vec3 reflectDir = reflect(-lightDir, vNormal);
        float spec = pow(max(dot(viewDir, reflectDir), 0.0), 64.0);

        float rim = 1.0 - max(dot(viewDir, normalize(vNormal)), 0.0);
        rim = pow(rim, 3.0);

        // Hologram color: darker base, hover makes lighter
        vec3 holoBaseDark = mix(vec3(0.0, 0.5, 0.5), vec3(0.3, 0.0, 0.4), hovered);
        vec3 holoBaseLight = mix(holoBaseDark, vec3(0.0, 0.7, 1.0), hovered);
        vec3 holoGlow = vec3(0.0, 0.5, 1.0) * rim * mix(1.2, 3.0, hovered); // increased glow multiplier

        // Scanline effect
        float scan = 0.5 + 0.4 * sin((vPosition.y + vPosition.x * 0.2) * 80.0 + mod(gl_FragCoord.y, 2.0) * 3.14);
        float scanStrength = mix(0.15, 0.35, hovered);

        // Flicker effect
        float flicker = 0.92 + 0.08 * sin(gl_FragCoord.y * 12.0 + gl_FragCoord.x * 0.5 + mod(gl_FragCoord.x + gl_FragCoord.y, 10.0));

        // Additive glow on hover
        float glow = pow(rim, 2.0) * mix(0.35, 1.2, hovered); // increased glow

        // Combine all
        vec3 color = holoBaseLight * (0.5 + 0.2 * diff) + holoGlow + spec * vec3(0.3, 0.7, 1.0);
        color = color * scan * scanStrength * flicker + glow;

        // Clamp for safety
        color = min(color, vec3(1.0));

        gl_FragColor = vec4(color, 0.7 + 0.25 * hovered); // slightly more opaque for stronger glow
    }
  `;

  const hovered = new Tween(0.3, { easing: quadOut });

  function handleEnter() {
    hovered.set(0.3, {
        duration: 0
      }).then(() => {hovered.set(1.0, { duration: 300 })});
  }

  function handleLeave() {
    hovered.set(0.3, { duration: 300 });
  }
</script>

<T.Mesh
  is={node}
  onpointerover={handleEnter}
  onpointerout={handleLeave}
>
  <T.ShaderMaterial
  {vertexShader}
  {fragmentShader}
  uniforms={{
    hovered: { value: 0.3 }
  }}
  uniforms.hovered.value={hovered.current}
/>
</T.Mesh>
