<script lang="ts">
	import '../app.css';
	
	let { children } = $props();
</script>

<div class="bg-gray-950 min-h-screen">
	{@render children()}
</div>
