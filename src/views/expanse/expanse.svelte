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
    ADD_EXPANSE_DATA,
    GET_CATEGORY_LIST,
    GET_EXPANSE_LIST,
    EDIT_EXPANSE_DATA,
    DELETE_EXPANSE_DATA,
  } from "../../routes/api_routes";
  import { getUserDetails } from "../../utils/storage/user";
  import { message } from "antd";
  import { onMount } from "svelte";
  import { LightPaginationNav } from "svelte-paginate";
  let expanse_data = {
    amount: "",
    description: "",
    category_id: "",
    date: "",
    other_category: "",
    id: "",
  };
  let expanse_data_error = {
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
  let open_expanse_modal = false;
  let expanseListData = [];
  let limit = 10;
  let page = 1;
  let total_page = 0;
  let total_records = 0;
  let delete_modal_open = false;
  let delete_id;
  const handleAddexpanseModal = () => {
    open_expanse_modal = true;
    expanse_data = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
      id: "",
    };
    expanse_data_error = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
    };
  };
  const handleAddexpanseData = async (event) => {
    isLoader = true;
    const isFormValid = validateForm();
    if (isFormValid) {
      if (other_category_show) {
        await post(ADD_CATEGORY_DATA, {
          user_id: parseInt(userData.id),
          name: expanse_data.other_category,
          type: "expanse",
        })
          .then((response) => {
            if (response.data.success) {
              expanse_data = {
                ...expanse_data,
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
        amount: expanse_data.amount,
        description: expanse_data.description,
        category_id: expanse_data.category_id,
        date: expanse_data.date,
        user_id: userData.id,
      };
      await post(ADD_EXPANSE_DATA, data)
        .then((response) => {
          if (response.data.success) {
            message.success(response.data.message);
            isLoader = false;
            expanse_data = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
              id: "",
            };
            expanse_data_error = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
            };
            other_category_show = false;
            open_expanse_modal = false;
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
      expanse_data_error = expanse_data_error;
    }
  };

  const validateForm = () => {
    let hasErrors = false; // Flag to track if there are any errors

    for (const key in expanse_data) {
      if (expanse_data.hasOwnProperty(key)) {
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
          expanse_data[key],
          type,
          maxLength,
          minLength
        );

        // Update error message in the expanse_data_error object
        expanse_data_error[key] = error.getError ? error.errorMsg : "";

        // Update hasErrors flag if there is an error
        if (error.getError) {
          hasErrors = true;
        }
      }
    }

    return !hasErrors; // Return true if no errors, false otherwise
  };

  const handleCancel = () => {
    open_expanse_modal = false;
    other_category_show = false;
    expanse_data = {
      amount: "",
      description: "",
      category_id: "",
      date: "",
      other_category: "",
      id: "",
    };
    expanse_data_error = {
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
    expanse_data = {
      ...expanse_data,
      [e.target.name]: value,
    };

    expanse_data_error = {
      ...expanse_data_error,
      [e.target.name]: error.getError ? error.errorMsg : "",
    };
  };

  const checkValidation = (field, value, type, maxLength, minLength) => {
    return fieldValidator(field, value, type, maxLength, minLength);
  };

  const handleKeyDown = (event) => {
    if (event.key === "Enter" || event.key === "enter") {
      handleAddexpanseData(event);
    }
  };

  onMount(async () => {
    getCategorysList();
    getexpanseListData(page);
    isLoader = true;
  });
  const getCategorysList = async () => {
    await get(GET_CATEGORY_LIST + userData.id + "/expanse").then((response) => {
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
    expanse_data = {
      ...expanse_data,
      [e.target.name]: parseInt(e.target.value),
    };
    expanse_data_error = {
      ...expanse_data_error,
      [e.target.name]: "",
    };
  };

  const handleDateChange = (e) => {
    expanse_data = {
      ...expanse_data,
      [e.target.name]: e.target.value,
    };
    expanse_data_error = {
      ...expanse_data_error,
      [e.target.name]: "",
    };
  };

  const getexpanseListData = async (page) => {
    await get(GET_EXPANSE_LIST + userData.id + "?page=" + page).then(
      (response) => {
        if (response.data.success) {
          expanseListData = response.data.data.records;
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
    await del(DELETE_EXPANSE_DATA + delete_id).then((response) => {
      if (response.data.success) {
        delete_modal_open = false;
        getexpanseListData((page = 1));
        message.success(response.data.message);
      } else {
        isLoader = false;
        message.error(response.data.message);
      }
    });
  };

  const handleEditeModal = (item) => {
    expanse_data = {
      ...expanse_data,
      amount: item.amount,
      description: item.description,
      category_id: item.category_id,
      date: item.date,
      other_category: "",
      id: item.id,
    };
    open_expanse_modal = true;
  };

  const handleEditexpanseData = async (event) => {
    isLoader = true;
    const isFormValid = validateForm();
    if (isFormValid) {
      if (other_category_show) {
        await post(ADD_CATEGORY_DATA, {
          user_id: parseInt(userData.id),
          name: expanse_data.other_category,
          type: "expanse",
        })
          .then((response) => {
            if (response.data.success) {
              expanse_data = {
                ...expanse_data,
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
        amount: expanse_data.amount,
        description: expanse_data.description,
        category_id: expanse_data.category_id,
        date: expanse_data.date,
        user_id: userData.id,
      };
      await put(EDIT_EXPANSE_DATA + expanse_data.id, data)
        .then((response) => {
          if (response.data.success) {
            message.success(response.data.message);
            isLoader = false;
            expanse_data = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
              id: "",
            };
            expanse_data_error = {
              amount: "",
              description: "",
              category_id: "",
              date: "",
              other_category: "",
            };
            other_category_show = false;
            open_expanse_modal = false;
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
      expanse_data_error = expanse_data_error;
    }
  };

  const getPageData = (pageValue) => {
    isLoader = true;
    page = pageValue;
    getexpanseListData(pageValue);
  };
</script>

<PrivateLayout>
  <header
    class="w-full bg-gray-200 text-black py-4 flex justify-between items-center"
  >
    <div class="flex items-center h-8 w-8 ml-4 font-semibold text-lg">
      Expanse
    </div>
    <!-- Logout button -->

    <button
      class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded inline-flex items-center mr-5"
      on:click={handleAddexpanseModal}
    >
      <PlusOutlined class="mr-2" />
      <span>Add Expanse</span>
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
          {#each expanseListData as item}
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
  isOpen={open_expanse_modal}
  onClose={handleCancel}
  handleSubmit={expanse_data.id
    ? (event) => handleEditexpanseData(event)
    : (event) => handleAddexpanseData(event)}
  submit_text={expanse_data.id ? "Update" : "Add"}
  modal_title={expanse_data.id ? "Edit Expanse Data" : "Add Expanse Data"}
>
  <form
    class="mb-4 max-w-md rounded bg-white px-8 pb-8 pt-6 shadow-md"
    on:submit={expanse_data.id
      ? (event) => handleEditexpanseData(event)
      : (event) => handleAddexpanseData(event)}
  >
    <div class="mb-4">
      <Input
        lable_class="block text-gray-700 text-sm font-bold mb-2"
        input_name="amount"
        lable_name="Amount"
        input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        input_type="text"
        placeholder="Enter amount."
        input_value={expanse_data.amount}
        isRequired="true"
        error={expanse_data_error.amount}
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
        input_value={expanse_data.description}
        isRequired="true"
        error={expanse_data_error.description}
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
        input_value={expanse_data.category_id}
        isRequired="true"
        error={expanse_data_error.category_id}
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
          input_value={expanse_data.other_category}
          isRequired="true"
          error={expanse_data_error.other_category}
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
        input_value={expanse_data.date}
        isRequired="true"
        error={expanse_data_error.date}
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
    Are you sure you want to delete this expanse information?
  </p>
</Modal>

<style>
</style>
