<script>
  // Your script goes here

  import PrivateLayout from "../../layouts/PrivateLayout.svelte";
  import {
    DeleteOutlined,
    EditOutlined,
    PlusOutlined,
  } from "svelte-ant-design-icons";
  import Modal from "../../components/Model.svelte";
  import Input from "../../components/Input.svelte";
  import { fieldValidator } from "../../utils/validation/custom";
  import Select from "../../components/Select.svelte";
  import { get, post, del, put } from "../../utils/api";
  import {
    ADD_CATEGORY_DATA,
    ADD_INCOME_DATA,
    GET_CATEGORY_LIST,
    GET_INCOME_LIST,
    EDIT_INCOME_DATA,
    DELETE_INCOME_DATA,
  } from "../../routes/api_routes";
  import { getUserDetails } from "../../utils/storage/user";
  import { message } from "antd";
  import { onMount } from "svelte";
  import { LightPaginationNav } from "svelte-paginate";
  let income_data = {
    amount: "",
    description: "",
    category_id: "",
    date: "",
    other_category: "",
    id: "",
  };
  let income_data_error = {
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
  let open_income_modal = false;
  let incomeListData = [];
  let limit = 10;
  let page = 1;
  let total_page = 0;
  let total_records = 0;
  let delete_modal_open = false;
  let delete_id;
  const handleAddIncomeModal = () => {
    open_income_modal = true;
    income_data = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
      id: "",
    };
    income_data_error = {
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
          user_id: parseInt(userData.id),
          name: income_data.other_category,
          type: "income",
        })
          .then((response) => {
            if (response.data.success) {
              income_data = {
                ...income_data,
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
        amount: income_data.amount,
        description: income_data.description,
        category_id: income_data.category_id,
        date: income_data.date,
        user_id: userData.id,
      };
      await post(ADD_INCOME_DATA, data)
        .then((response) => {
          if (response.data.success) {
            message.success(response.data.message);
            isLoader = false;
            income_data = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
              id: "",
            };
            income_data_error = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
            };
            other_category_show = false;
            open_income_modal = false;
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
      income_data_error = income_data_error;
    }
  };

  const validateForm = () => {
    let hasErrors = false; // Flag to track if there are any errors

    for (const key in income_data) {
      if (income_data.hasOwnProperty(key)) {
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
          income_data[key],
          type,
          maxLength,
          minLength
        );

        // Update error message in the income_data_error object
        income_data_error[key] = error.getError ? error.errorMsg : "";

        // Update hasErrors flag if there is an error
        if (error.getError) {
          hasErrors = true;
        }
      }
    }

    return !hasErrors; // Return true if no errors, false otherwise
  };

  const handleCancel = () => {
    open_income_modal = false;
    other_category_show = false;
    income_data = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
      id: "",
    };
    income_data_error = {
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
    income_data = {
      ...income_data,
      [e.target.name]: value,
    };

    income_data_error = {
      ...income_data_error,
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
    getIncomeListData(page);
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
    income_data = {
      ...income_data,
      [e.target.name]: parseInt(e.target.value),
    };
    income_data_error = {
      ...income_data_error,
      [e.target.name]: "",
    };
  };

  const handleDateChange = (e) => {
    income_data = {
      ...income_data,
      [e.target.name]: e.target.value,
    };
    income_data_error = {
      ...income_data_error,
      [e.target.name]: "",
    };
  };

  const getIncomeListData = async (page) => {
    await get(GET_INCOME_LIST + userData.id + "?page=" + page).then(
      (response) => {
        if (response.data.success) {
          incomeListData = response.data.data.records;
          limit = response.data.data.limit;
          page = response.data.data.page;
          total_page = response.data.data.total_page;
          total_records = response.data.data.total_records;

          isLoader = false;
          message.success(response.data.message);
        } else {
          isLoader = false;
          message.error(response.data.message);
        }
      }
    );
  };

  const handleDeleteModal = (item) => {
    delete_id = item.id;
    delete_modal_open = true;
  };

  const handleDeleteCancel = () => {
    delete_modal_open = false;
  };

  const handleDelete = async (event) => {
    isLoader = true;
    await del(DELETE_INCOME_DATA + delete_id).then((response) => {
      if (response.data.success) {
        delete_modal_open = false;
        getIncomeListData((page = 1));
        message.success(response.data.message);
      } else {
        isLoader = false;
        message.error(response.data.message);
      }
    });
  };

  const handleEditeModal = (item) => {
    income_data = {
      ...income_data,
      amount: item.amount,
      description: item.description,
      category_id: item.category_id,
      date: item.date,
      other_category: "",
      id: item.id,
    };
    open_income_modal = true;
  };

  const handleEditIncomeData = async (event) => {
    isLoader = true;
    const isFormValid = validateForm();
    if (isFormValid) {
      if (other_category_show) {
        await post(ADD_CATEGORY_DATA, {
          user_id: parseInt(userData.id),
          name: income_data.other_category,
          type: "income",
        })
          .then((response) => {
            if (response.data.success) {
              income_data = {
                ...income_data,
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
        amount: income_data.amount,
        description: income_data.description,
        category_id: income_data.category_id,
        date: income_data.date,
        user_id: userData.id,
      };
      await put(EDIT_INCOME_DATA + income_data.id, data)
        .then((response) => {
          if (response.data.success) {
            message.success(response.data.message);
            isLoader = false;
            income_data = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
              id: "",
            };
            income_data_error = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
            };
            other_category_show = false;
            open_income_modal = false;
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
      income_data_error = income_data_error;
    }
  };

  const getPageData = (pageValue) => {
    isLoader = true;
    page = pageValue;
    getIncomeListData(pageValue);
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

  <main class="m-5">
    <div class="overflow-x-auto">
      <table
        class="table-auto min-w-full bg-white shadow-md rounded-lg overflow-hidden"
      >
        <thead class="bg-gray-200 text-gray-700">
          <tr>
            <th class="px-4 py-2">Amount</th>
            <th class="px-4 py-2">Description</th>
            <th class="px-4 py-2">Date</th>
            <th class="px-4 py-2">Action</th>
          </tr>
        </thead>
        <tbody class="text-gray-600">
          {#each incomeListData as item}
            <tr class="hover:bg-gray-100 transition-colors duration-200">
              <td class="border px-4 py-2">{item.amount}</td>
              <td class="border px-4 py-2">{item.description || "NA"}</td>
              <td class="border px-4 py-2"
                >{new Date(item.date).toLocaleDateString()}</td
              >
              <td class="border px-4 py-2 flex items-center">
                <button
                  class="flex items-center text-blue-500 hover:text-blue-700 mr-2"
                  on:click={() => handleEditeModal(item)}
                >
                  <EditOutlined class=" mr-1" />
                </button>
                <button
                  class="flex items-center text-red-500 hover:text-red-700"
                  on:click={() => handleDeleteModal(item)}
                >
                  <DeleteOutlined class=" mr-1" />
                </button>
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </main>

  <div class="d-flex justify-content-end flex-row w-100">
    <LightPaginationNav
      totalItems={total_records}
      pageSize={limit}
      currentPage={page}
      {limit}
      showStepOptions={true}
      on:setPage={(e) => getPageData(e.detail.page)}
    />
  </div>
</PrivateLayout>

<Modal
  isOpen={open_income_modal}
  onClose={handleCancel}
  handleSubmit={income_data.id
    ? (event) => handleEditIncomeData(event)
    : (event) => handleAddIncomeData(event)}
  submit_text={income_data.id ? "Update" : "Add"}
  modal_title={income_data.id ? "Edit Income Data" : "Add Income Data"}
>
  <form
    class="mb-4 max-w-md rounded bg-white px-8 pb-8 pt-6 shadow-md"
    on:submit={income_data.id
      ? (event) => handleEditIncomeData(event)
      : (event) => handleAddIncomeData(event)}
  >
    <div class="mb-4">
      <Input
        lable_class="block text-gray-700 text-sm font-bold mb-2"
        input_name="amount"
        lable_name="Amount"
        input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        input_type="text"
        placeholder="Enter amount."
        input_value={income_data.amount}
        isRequired="true"
        error={income_data_error.amount}
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
        input_value={income_data.description}
        isRequired="true"
        error={income_data_error.description}
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
        input_value={income_data.category_id}
        isRequired="true"
        error={income_data_error.category_id}
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
          input_value={income_data.other_category}
          isRequired="true"
          error={income_data_error.other_category}
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
        input_value={income_data.date}
        isRequired="true"
        error={income_data_error.date}
        handleChange={(e) => handleDateChange(e)}
        {handleKeyDown}
      />
    </div>
  </form>
</Modal>

<Modal
  isOpen={delete_modal_open}
  onClose={handleDeleteCancel}
  handleSubmit={(event) => handleDelete(event)}
  submit_text="Delete"
  modal_title="Confirmation for delete"
>
  <p class="text-gray-700">
    Are you sure you want to delete this income information?
  </p>
</Modal>

<style>
</style>
