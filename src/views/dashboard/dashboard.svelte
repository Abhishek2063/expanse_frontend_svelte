<script>
  import PrivateLayout from "../../layouts/PrivateLayout.svelte";

  // Your script goes here
</script>

<PrivateLayout>
  <div class="message-container">
    <p class="message">Coming soon...</p>
  </div>
</PrivateLayout>

<style>
  /* Your styles go here */
  .message-container {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
  }

  .message {
    font-size: 24px;
  }
</style>
