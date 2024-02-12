<script>
  export let text;
  export let class_values;
  export let type;
  export let handleClick;
</script>

<button class={class_values} {type} on:click={handleClick}>{text}</button>

<style>
</style>
