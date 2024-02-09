<script>
  import { private_route, public_routes } from "../routes/app_routes";
  import logo_image from "../assets/images/logo.png";
  import { navigate } from "svelte-routing";
  import { LogoutOutlined } from "svelte-ant-design-icons";
  import { clearUserDetails, getUserDetails } from "../utils/storage/user";
  import { post } from "../utils/api";
  import { LOGOUT_API } from "../routes/api_routes";
  import { message } from "antd";
  import { removeLocalData } from "../utils/storage/token";
  // Define the list of pages with their respective paths
  const pages = [
    { name: "Dashboard", path: private_route.dashboard },
    { name: "Income", path: private_route.income },
    { name: "Expense", path: private_route.expense },
    { name: "Profile", path: private_route.profile },
  ];
  let isLoader = false;

  // Get the current page's path
  let currentPage = window.location.pathname;

  const handleNavigate = (e, path) => {
    e.preventDefault();
    currentPage = path;
    navigate(path);
  };

  const handleLogout = async () => {
    isLoader = true;
    const userData = getUserDetails();
    await post(LOGOUT_API + "/" + userData.id)
      .then((response) => {
        if (response.data.success) {
          message.success(response.data.message);
          clearUserDetails();
          removeLocalData();
          navigate(public_routes.login);
          isLoader = false;
        } else {
          isLoader = false;
          message.error(response.data.message);
        }
      })
      .catch((error) => {
        isLoader = false;
        message.error(error.response.data.message);
      });
  };
</script>

<header
  class="fixed top-0 w-full bg-gray-800 text-white py-4 flex justify-between items-center"
>
  <div class="flex items-center">
    <!-- Logo image -->
    <img src={logo_image} alt="Logo" class="h-8 w-8 ml-4" />

    <!-- Page names -->
    {#each pages as page}
      <button
        class={currentPage === page.path
          ? "ml-4 text-lg font-semibold border-b-2 border-white"
          : "ml-4 text-lg font-semibold"}
        type="button"
        on:click={(e) => handleNavigate(e, page.path)}
      >
        {page.name}
      </button>
    {/each}
  </div>
  <div class="flex items-center mr-4">
    <!-- Logout button -->
    <button class="text-white" on:click={handleLogout}>
      <LogoutOutlined />
    </button>
  </div>
</header>
