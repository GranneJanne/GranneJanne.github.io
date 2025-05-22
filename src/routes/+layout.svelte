<script lang="ts">
	import '../app.css';
	
	let { children } = $props();
</script>

<div id="main">
	{@render children()}
</div>

<style>
  div#main {
    height: 100%;
	background-color: rgb(0, 0, 10);
  }
</style>