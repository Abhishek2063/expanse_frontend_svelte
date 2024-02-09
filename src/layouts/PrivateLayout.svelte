<!-- layouts/PrivateLayout.svelte -->
<script>
  import Header from "../components/Header.svelte";
  import Footer from "../components/Footer.svelte";
</script>

<Header />
<main class="mt-16">
  <slot />
</main>
<Footer />

<style>
  main {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
  }
</style>
