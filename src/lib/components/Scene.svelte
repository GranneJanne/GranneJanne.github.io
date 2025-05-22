<script>
  import { T } from "@threlte/core";
  import { OrbitControls, useGltf, interactivity } from "@threlte/extras";
  import BrainPart from './BrainPart.svelte';

  interactivity({
    filter: (hits) => {
      return hits ? [hits[0]] : [];
    }
  });
  const brain = useGltf("/models/brain2.glb");
  let autoRotate = true;
</script>

<T.PerspectiveCamera makeDefault position={[0, 0, 7]} oncreate={(ref) => ref.lookAt(0, 0, 0)}>
  <OrbitControls
    target={[0, 0, 0]}
    enablePan={false}
    enableZoom={false}
    enableDamping={true}
    dampingFactor={0.1}
    autoRotate={autoRotate}
    minPolarAngle={Math.PI / 2.8}
    maxPolarAngle={Math.PI / 2.8}
  />
</T.PerspectiveCamera>

{#if $brain?.nodes}
  {#each Object.values($brain.nodes).slice(1) as node}
    <BrainPart {node} />
  {/each}
{/if}

