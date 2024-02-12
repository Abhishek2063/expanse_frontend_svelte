<script>
  export let isOpen = false;
  export let onClose;
  export let handleSubmit;
  export let submit_text;
  export let modal_title;

  import { CloseOutlined } from "svelte-ant-design-icons";
  function closeModal() {
    onClose();
  }

  function addModal() {
    handleSubmit();
  }
</script>

<div class="modal" class:open={isOpen}>
  <div class="popup">
    <!-- Header -->
    <div class="popup-header flex justify-between mb-4">
      <!-- Title -->
      <h2 class="text-lg font-semibold">{modal_title}</h2>
      <!-- Close button -->
      <button on:click={closeModal}><CloseOutlined /></button>
    </div>
    <!-- Body (Content) -->
    <div class="popup-content mb-4">
      <slot />
    </div>
    <!-- Footer -->
    <div class="popup-footer flex justify-end">
      <!-- Close button (again) -->
      <button class="close-button mr-2" on:click={closeModal}>Close</button>

      <!-- Add button -->
      <button class="add-button mr-2" on:click={addModal}>{submit_text}</button>
    </div>
  </div>
</div>

<style>
  .modal {
    display: none;
    align-items: center;
    justify-content: center;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 9999;
  }

  .open {
    display: flex;
  }

  .popup {
    background-color: #fff;
    padding: 20px;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    max-width: 400px;
  }

  .popup-header {
    margin-bottom: 10px;
  }

  .popup-content {
    margin-bottom: 10px;
  }

  .popup-footer {
    margin-top: 10px;
  }

  .close-button {
    padding: 8px 16px;
    border-radius: 4px;
    background-color: #e53935;
    color: #fff;
    border: none;
    font-size: 14px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .close-button:hover {
    background-color: #c62828;
  }

  .add-button {
    padding: 8px 16px;
    border-radius: 4px;
    background-color: #18d5f7cc;
    color: #fff;
    border: none;
    font-size: 14px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .add-button:hover {
    background-color: #15fdcb;
  }
</style>
