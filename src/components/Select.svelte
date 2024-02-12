<script>
  export let lable_class;
  export let input_name;
  export let lable_name;
  export let input_class;
  export let input_options;
  export let input_value;
  export let isRequired;
  export let error;
  export let handleSelectChange;
  export let placeholder;
  export let handleKeyDown;
</script>

<label class={lable_class} for={input_name}>
  {#if isRequired}
    {lable_name}
    <span class="text-red-700">*</span>
  {:else}
    {lable_name}
  {/if}
</label>
<select
  class={input_class}
  id={input_name}
  name={input_name}
  value={input_value}
  {placeholder}
  on:change={handleSelectChange}
  on:keydown={handleKeyDown}
>
  {#if input_options.length === 0}
    <option value="" disabled>No options available</option>
  {:else}
    {#each input_options as option}
      <option value={option.value}>{option.label}</option>
    {/each}
  {/if}
</select>
{#if error}
  <p><small style="color: red">{error}</small></p>
{/if}

<style>
</style>
