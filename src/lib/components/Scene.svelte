<script>
  import { T } from "@threlte/core";
  import { OrbitControls, useGltf, interactivity } from "@threlte/extras";
  import BrainPart from './BrainPart.svelte';

  interactivity();
  const brain = useGltf("/models/brain2.glb");
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 8]} oncreate={(ref) => ref.lookAt(0, 0, 0)}>
  <OrbitControls
    target={[0, 0, 0]}
    enablePan={false}
    enableZoom={false}
    autoRotate={true}
    minPolarAngle={Math.PI / 3}
    maxPolarAngle={Math.PI / 3}
  />
</T.PerspectiveCamera>

{#if $brain?.nodes}
  {#each Object.values($brain.nodes).slice(1) as node}
    <BrainPart {node} />
  {/each}
{/if}

