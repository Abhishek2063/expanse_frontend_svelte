<script>
	export let text;
	export let class_values;
	export let navigate_path;
	import { navigate } from 'svelte-routing'
 const handleClick = () => {
	navigate(navigate_path)
 }
</script>

<button class={class_values} on:click={handleClick}>
	{text}
</button>

<style>
</style>
