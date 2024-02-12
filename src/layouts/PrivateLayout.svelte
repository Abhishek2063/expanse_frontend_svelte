<!-- layouts/PrivateLayout.svelte -->
<script>
  import Header from "../components/Header.svelte";
  import Footer from "../components/Footer.svelte";

  let isLoader = false;
</script>

<Header {isLoader} />
<main class="mt-16">
  <slot />
</main>
<Footer />

<style>
 
</style>
