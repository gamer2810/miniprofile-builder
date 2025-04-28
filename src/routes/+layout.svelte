

<script lang="ts">
	export const prerender = true;
	import '../app.css';
	
	let { children } = $props();
</script>

{@render children()}
