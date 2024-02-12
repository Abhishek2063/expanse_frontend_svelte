<script>
  // Your script goes here

  import PrivateLayout from "../../layouts/PrivateLayout.svelte";
  import { getUserDetails } from "../../utils/storage/user";

  const userData = getUserDetails();

  // Function to generate random background color
  function getRandomColor() {
    const letters = "0123456789ABCDEF";
    let color = "#";
    for (let i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }

  // Extract initials from first and last name
  const initials = `${userData.first_name.charAt(0)}${userData.last_name.charAt(0)}`;
  
  // Generate random background color for the initials
  const bgColor = getRandomColor();
</script>

<PrivateLayout>
  <div class="flex justify-center items-center message-container">
    <div class="bg-white rounded-lg shadow-lg flex items-center justify-center  h-44 w-full">
      <!-- Display profile image with initials and background color -->
      <div class="profile-image" style="background-color: {bgColor}">
        <span>{initials}</span>
      </div>
      <!-- Display full name and email -->
      <div class="ml-5">
        <h2 class="text-2xl font-semibold">{userData.first_name} {userData.last_name}</h2>
        <p class="text-gray-600">{userData.email}</p>
      </div>
    </div>
  </div>
</PrivateLayout>

<style>
  /* Your styles go here */
  .profile-image {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 48px;
    color: #ffffff;
  }

  .profile-image span {
    text-transform: uppercase;
  }
  .message-container {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

</style>
