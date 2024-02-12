<script>
  import { fieldValidator } from "./../../utils/validation/custom.js";
  import { message } from "antd";
  import { DASHBOARD_DATA } from "../../routes/api_routes";
  import { get } from "../../utils/api";
  import { getUserDetails } from "../../utils/storage/user";
  import { onMount } from "svelte";
  import PrivateLayout from "../../layouts/PrivateLayout.svelte";
  import PieChart from "../../components/PieChart.svelte";
  import Loader from "../../utils/loader.svelte";
  import Input from "../../components/Input.svelte";

  // Your script goes here
  const userData = getUserDetails();
  let dashboard_data = {};
  let dashboard_pie_data = [];
  let isLoader = false;
  let dashboardData = {
    start_date: "",
    end_date: "",
  };
  let dashboardDataError = {
    start_date: "",
    end_date: "",
  };
  onMount(async () => {
    isLoader = true;
    await getDashboardData();
  });

  const getDashboardData = async () => {
    try {
      const response = await get(
        DASHBOARD_DATA +
          userData.id +
          "?start_date=" +
          dashboardData.start_date +
          "&end_date=" +
          dashboardData.end_date
      );
      if (response.data.success) {
        dashboard_pie_data = [
          { name: "Balance", value: response.data.data.balance },
          {
            name: "Total Expense",
            value: response.data.data.total_expense
              ? response.data.data.total_expense
              : 0,
          },
          {
            name: "Total Income",
            value: response.data.data.total_income
              ? response.data.data.total_income
              : 0,
          },
        ];
        dashboard_data = response.data.data;
        isLoader = false;
        message.success(response.data.message);
      } else {
        isLoader = false;
        message.error(response.data.message);
      }
    } catch (error) {
      isLoader = false;
      message.error("Failed to fetch dashboard data");
    }
  };

  const handleDateChange = (e) => {
    dashboardData = {
      ...dashboardData,
      [e.target.name]: e.target.value,
    };
    dashboardDataError = {
      ...dashboardDataError,
      [e.target.name]: "",
    };
  };

  const handleKeyDown = (event) => {
    if (event.key === "Enter" || event.key === "enter") {
      handleSubmit(event);
    }
  };

  const handleSubmit = (event) => {
    event.preventDefault();
    isLoader = true;
    const isFormValid = validateForm();
    if (isFormValid) {
      getDashboardData();
    } else {
      isLoader = false;
      dashboardDataError = dashboardDataError;
    }
  };
  const validateForm = () => {
    let hasErrors = false; // Flag to track if there are any errors

    for (const key in dashboardData) {
      if (dashboardData.hasOwnProperty(key)) {
        let type = "";
        let maxLength = null;
        let minLength = null;

        if (key === "start_date") {
          type = "required";
          maxLength = null;
          minLength = null;
        }
        if (key === "end_date") {
          type = "required";
          maxLength = null;
          minLength = null;
        }
        const error = checkValidation(
          key,
          dashboardData[key],
          type,
          maxLength,
          minLength
        );

        // Update error message in the dashboardDataError object
        dashboardDataError[key] = error.getError ? error.errorMsg : "";

        // Update hasErrors flag if there is an error
        if (error.getError) {
          hasErrors = true;
        }
      }
    }

    return !hasErrors; // Return true if no errors, false otherwise
  };

  const checkValidation = (field, value, type, maxLength, minLength) => {
    return fieldValidator(field, value, type, maxLength, minLength);
  };
</script>

{#if isLoader}
  <Loader />
{:else}
  <PrivateLayout>
    <header
      class="w-full bg-gray-200 text-black py-4 flex justify-between items-center"
    >
      <div class="flex items-center h-8 w-8 ml-4 font-semibold text-lg">
        Dashboard
      </div>
      <!-- Logout button -->
    </header>

    <div class="w-full py-4 flex justify-between items-center">
      <div class="flex justify-start items-center space-x-4 w-1/2">
        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="start_date"
          lable_name="Start Date"
          input_class="shadow appearance-none border rounded py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="date"
          placeholder="Enter start date."
          input_value={dashboardData.start_date}
          isRequired="true"
          handleChange={(e) => handleDateChange(e)}
          error={dashboardDataError.start_date}
          {handleKeyDown}
        />

        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="end_date"
          lable_name="End Date"
          input_class="shadow appearance-none border rounded py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="date"
          placeholder="Enter end date."
          input_value={dashboardData.end_date}
          isRequired="true"
          handleChange={(e) => handleDateChange(e)}
          error={dashboardDataError.end_date}
          {handleKeyDown}
        />
      </div>

      <button
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 mr-5 rounded"
        on:click={(event) => handleSubmit(event)}
      >
        Submit
      </button>
    </div>

    <div class="flex justify-between pt-16">
      <div class="w-1/2 px-4">
        <div class="bg-white shadow rounded-lg p-4 mb-4">
          <h2 class="text-lg font-bold mb-2">Total Income</h2>
          <p class="text-xl font-semibold text-green-500">
            {dashboard_data.total_income}
          </p>
        </div>
        <div class="bg-white shadow rounded-lg p-4 mb-4">
          <h2 class="text-lg font-bold mb-2">Total Expense</h2>
          <p class="text-xl font-semibold text-red-500">
            {dashboard_data.total_expense}
          </p>
        </div>
        <div class="bg-white shadow rounded-lg p-4 mb-4">
          <h2 class="text-lg font-bold mb-2">Total Balance</h2>
          <p class="text-xl font-semibold">{dashboard_data.balance}</p>
        </div>
      </div>
      <div class="w-1/2 px-4">
        <PieChart data={dashboard_pie_data} />
      </div>
    </div>
  </PrivateLayout>
{/if}

<style>
  /* Your styles go here */
</style>
