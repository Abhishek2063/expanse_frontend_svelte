<script>
  // Your script goes here

  import PrivateLayout from "../../layouts/PrivateLayout.svelte";
  import { PlusOutlined } from "svelte-ant-design-icons";
  import Modal from "../../components/Model.svelte";
  import Input from "../../components/Input.svelte";
  import { fieldValidator } from "../../utils/validation/custom";
  import Select from "../../components/Select.svelte";
  import { get, post } from "../../utils/api";
  import {
    ADD_CATEGORY_DATA,
    ADD_INCOME_DATA,
    GET_CATEGORY_LIST,
  } from "../../routes/api_routes";
  import { getUserDetails } from "../../utils/storage/user";
  import { message } from "antd";
  import { onMount } from "svelte";
  let add_income_data = {
    amount: "",
    description: "",
    category_id: "",
    date: "",
    other_category: "",
  };
  let add_income_data_error = {
    amount: "",
    description: "",
    category_id: "",
    date: "",
    other_category: "",
  };
  const userData = getUserDetails();
  let other_category_show = false;
  let isLoader = false;
  let category_list = [];
  let open_add_icome_modal = false;
  const handleAddIncomeModal = () => {
    open_add_icome_modal = true;
    add_income_data = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
    };
    add_income_data_error = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
    };
  };
  const handleAddIncomeData = async (event) => {

    isLoader = true;
    const isFormValid = validateForm();
    if (isFormValid) {
      if (other_category_show) {
        await post(ADD_CATEGORY_DATA, {
          user_id:parseInt( userData.id),
          name: add_income_data.other_category,
          type: "income",
        })
          .then((response) => {
            if (response.data.success) {
              add_income_data = {
                ...add_income_data,
                category_id: response.data.data.data.id,
              };
            } else {
              isLoader = false;
              message.error(response.data.message);
            }
          })
          .catch((error) => {
            isLoader = false;
            message.error(error.response.data.message);
          });
      }
      const data = {
        amount: add_income_data.amount,
        description: add_income_data.description,
        category_id: add_income_data.category_id,
        date: add_income_data.date,
        user_id: userData.id,
      };
      await post(ADD_INCOME_DATA, data)
        .then((response) => {
          if (response.data.success) {
            message.success(response.data.message);
            isLoader = false;
            add_income_data = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
            };
            add_income_data_error = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
            };
            other_category_show = false;
            open_add_icome_modal = false;
          } else {
            isLoader = false;
            message.error(response.data.message);
          }
        })
        .catch((error) => {
          isLoader = false;
          message.error(error.response.data.message);
        });
    } else {
      isLoader = false;
      add_income_data_error = add_income_data_error;
    }
  };

  const validateForm = () => {
    let hasErrors = false; // Flag to track if there are any errors

    for (const key in add_income_data) {
      if (add_income_data.hasOwnProperty(key)) {
        let type = "";
        let maxLength = null;
        let minLength = null;

        if (key === "amount") {
          type = "decimalNumber";
          maxLength = null;
          minLength = null;
        }
        if (key === "description") {
          type = "alphabetics";
          maxLength = 50;
          minLength = 2;
        }
        if (key === "category_id") {
          type = "required";
          maxLength = null;
          minLength = null;
        }
        if (other_category_show && key === "other_category") {
          type = "alphabetics";
          maxLength = 50;
          minLength = 2;
        }
        if (key === "date") {
          type = "required";
          maxLength = null;
          minLength = null;
        }
        const error = checkValidation(
          key,
          add_income_data[key],
          type,
          maxLength,
          minLength
        );

        // Update error message in the add_income_data_error object
        add_income_data_error[key] = error.getError ? error.errorMsg : "";

        // Update hasErrors flag if there is an error
        if (error.getError) {
          hasErrors = true;
        }
      }
    }

    return !hasErrors; // Return true if no errors, false otherwise
  };

  const handleCancel = () => {
    open_add_icome_modal = false;
    other_category_show = false;
    add_income_data = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
    };
    add_income_data_error = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
    };
  };

  const handleInputValue = (e, type, maxLength, minLength) => {
    let value = e.target.value;

    let error = checkValidation(
      e.target.name,
      value,
      type,
      maxLength,
      minLength
    );
    // Update form data and error messages
    add_income_data = {
      ...add_income_data,
      [e.target.name]: value,
    };

    add_income_data_error = {
      ...add_income_data_error,
      [e.target.name]: error.getError ? error.errorMsg : "",
    };
  };

  const checkValidation = (field, value, type, maxLength, minLength) => {
    return fieldValidator(field, value, type, maxLength, minLength);
  };

  const handleKeyDown = (event) => {
    if (event.key === "Enter" || event.key === "enter") {
      handleAddIncomeData(event);
    }
  };

  onMount(async () => {
    getCategorysList();
    isLoader = true;
  });
  const getCategorysList = async () => {
    await get(GET_CATEGORY_LIST + userData.id + "/income").then((response) => {
      if (response.data.success) {
        category_list = response.data.data.map((category) => ({
          value: category.id,
          label: category.name,
        }));
        category_list.push({ value: 0, label: "other_category" });
        isLoader = false;
      } else {
        isLoader = false;
        message.error(response.data.message);
      }
    });
  };

  const handleSelectChange = (e) => {
    if (e.target.value === "0" || e.target.value === 0) {
      other_category_show = true;
    }
    add_income_data = {
      ...add_income_data,
      [e.target.name]: parseInt(e.target.value),
    };
    add_income_data_error = {
      ...add_income_data_error,
      [e.target.name]: "",
    };
  };

  const handleDateChange = (e) => {
    add_income_data = {
      ...add_income_data,
      [e.target.name]: e.target.value,
    };
    add_income_data_error = {
      ...add_income_data_error,
      [e.target.name]: "",
    };
  };
</script>

<PrivateLayout>
  <header
    class="w-full bg-gray-200 text-black py-4 flex justify-between items-center"
  >
    <div class="flex items-center h-8 w-8 ml-4 font-semibold text-lg">
      Income
    </div>
    <!-- Logout button -->

    <button
      class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded inline-flex items-center mr-5"
      on:click={handleAddIncomeModal}
    >
      <PlusOutlined class="mr-2" />
      <span>Add Income</span>
    </button>
  </header>
</PrivateLayout>

<Modal
  isOpen={open_add_icome_modal}
  onClose={handleCancel}
  handleSubmit={(event) => handleAddIncomeData(event)}
  submit_text="Add"
  modal_title="Add Income Data"
>
  <form
    class="mb-4 max-w-md rounded bg-white px-8 pb-8 pt-6 shadow-md"
    on:submit={(event) => handleAddIncomeData(event)}
  >
    <div class="mb-4">
      <Input
        lable_class="block text-gray-700 text-sm font-bold mb-2"
        input_name="amount"
        lable_name="Amount"
        input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        input_type="text"
        placeholder="Enter amount."
        input_value={add_income_data.amount}
        isRequired="true"
        error={add_income_data_error.amount}
        handleChange={(e) => handleInputValue(e, "decimalNumber", null, null)}
        {handleKeyDown}
      />
    </div>

    <div class="mb-4">
      <Input
        lable_class="block text-gray-700 text-sm font-bold mb-2"
        input_name="description"
        lable_name="Description"
        input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        input_type="text"
        placeholder="Enter description."
        input_value={add_income_data.description}
        isRequired="true"
        error={add_income_data_error.description}
        handleChange={(e) => handleInputValue(e, "alphabetics", 50, 2)}
        {handleKeyDown}
      />
    </div>

    <div class="mb-4">
      <Select
        lable_class="block text-gray-700 text-sm font-bold mb-2"
        input_name="category_id"
        lable_name="Select Category"
        input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        placeholder="Enter select Category."
        input_value={add_income_data.category_id}
        isRequired="true"
        error={add_income_data_error.category_id}
        handleSelectChange={(e) => handleSelectChange(e)}
        {handleKeyDown}
        input_options={category_list}
      />
    </div>

    {#if other_category_show}
      <div class="mb-4">
        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="other_category"
          lable_name="Other Category"
          input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="text"
          placeholder="Enter other category."
          input_value={add_income_data.other_category}
          isRequired="true"
          error={add_income_data_error.other_category}
          handleChange={(e) => handleInputValue(e, "alphabetics", 50, 2)}
          {handleKeyDown}
        />
      </div>
    {/if}

    <div class="mb-4">
      <Input
        lable_class="block text-gray-700 text-sm font-bold mb-2"
        input_name="date"
        lable_name="Date"
        input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        input_type="date"
        placeholder="Enter date."
        input_value={add_income_data.date}
        isRequired="true"
        error={add_income_data_error.date}
        handleChange={(e) => handleDateChange(e)}
        {handleKeyDown}
      />
    </div>
  </form>
</Modal>

<style>
</style>
