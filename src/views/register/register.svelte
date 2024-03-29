<script>
  import { message } from "antd";
  import { REGISTER_API } from "../../routes/api_routes";
  import { post } from "../../utils/api";
  import { fieldValidator } from "../../utils/validation/custom";
  import Loader from "../../utils/loader.svelte";
  import Input from "../../components/Input.svelte";
  import Button from "../../components/Button.svelte";
  import { public_routes } from "../../routes/app_routes";
  import { navigate } from "svelte-routing";
  import NavigateButton from "../../components/NavigateButton.svelte";
  import PublicLayout from "../../layouts/PublicLayout.svelte";

  let register_form_data = {
    first_name: "",
    last_name: "",
    email: "",
    password: "",
    confirm_password: "",
  };

  let register_form_data_error = {
    first_name: "",
    last_name: "",
    email: "",
    password: "",
    confirm_password: "",
  };

  let isLoader = false;

  const validateForm = () => {
    let hasErrors = false; // Flag to track if there are any errors

    for (const key in register_form_data) {
      if (register_form_data.hasOwnProperty(key)) {
        let type = "";
        let maxLength = null;
        let minLength = null;

        if (key === "first_name" || key === "last_name") {
          type = "alphabetics";
          maxLength = 50;
          minLength = 2;
        }
        if (key === "email") {
          type = "email";
          maxLength = 255;
          minLength = 8;
        }
        if (key === "password" || key === "confirm_password") {
          type = "password";
          maxLength = 20;
          minLength = 8;
        }

        const error = checkValidation(
          key,
          register_form_data[key],
          type,
          maxLength,
          minLength
        );

        // Update error message in the register_form_data_error object
        register_form_data_error[key] = error.getError ? error.errorMsg : "";

        // Update hasErrors flag if there is an error
        if (error.getError) {
          hasErrors = true;
        }
      }
    }

    return !hasErrors; // Return true if no errors, false otherwise
  };

  const handleSubmit = async (event) => {
    event.preventDefault();

    isLoader = true;
    const isFormValid = validateForm();
    if (isFormValid) {
      await post(REGISTER_API, register_form_data)
        .then((response) => {
          if (response.data.success) {
            message.success(response.data.message);
            register_form_data = {
              first_name: "",
              last_name: "",
              email: "",
              password: "",
              confirm_password: "",
            };
            register_form_data_error = {
              first_name: "",
              last_name: "",
              email: "",
              password: "",
              confirm_password: "",
            };
            isLoader = false;
            navigate(public_routes.login);
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
      register_form_data_error = register_form_data_error;
    }
  };

  const handleKeyDown = (event) => {
    if (event.key === "Enter" || event.key === "enter") {
      handleSubmit(event);
    }
  };

  const checkValidation = (field, value, type, maxLength, minLength) => {
    return fieldValidator(
      field,
      value,
      type,
      maxLength,
      minLength,
      register_form_data.password
    );
  };

  const handleInputValue = (e, type, maxLength, minLength) => {
    let value = e.target.value;
    if (type === "password" || type === "email") {
      value = e.target.value.replace(/\s+/g, "");
    }
    let error = checkValidation(
      e.target.name,
      value,
      type,
      maxLength,
      minLength
    );
    // Update form data and error messages
    register_form_data = {
      ...register_form_data,
      [e.target.name]: value,
    };

    register_form_data_error = {
      ...register_form_data_error,
      [e.target.name]: error.getError ? error.errorMsg : "",
    };
  };
</script>

<PublicLayout>
  <!-- markup (zero or more items) goes here -->
  {#if isLoader}
    <Loader />
  {/if}
  <div class=" flex h-screen w-full items-center justify-center">
    <form
      class="mb-4 max-w-md rounded bg-white px-8 pb-8 pt-6 shadow-md"
      on:submit={handleSubmit}
    >
      <h1 class="mb-2 text-center text-lg font-bold text-cyan-500">
        Create an account?
      </h1>
      <div class="mb-4">
        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="first_name"
          lable_name="First Name"
          input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="text"
          placeholder="Enter first name."
          input_value={register_form_data.first_name}
          isRequired="true"
          error={register_form_data_error.first_name}
          handleChange={(e) => handleInputValue(e, "alphabetics", 50, 2)}
          {handleKeyDown}
        />
      </div>
      <div class="mb-4">
        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="last_name"
          lable_name="Last Name"
          input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="text"
          placeholder="Enter last name."
          input_value={register_form_data.last_name}
          isRequired="true"
          error={register_form_data_error.last_name}
          handleChange={(e) => handleInputValue(e, "alphabetics", 50, 2)}
          {handleKeyDown}
        />
      </div>
      <div class="mb-4">
        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="email"
          lable_name="Email"
          input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="email"
          placeholder="Enter email."
          input_value={register_form_data.email}
          isRequired="true"
          error={register_form_data_error.email}
          handleChange={(e) => handleInputValue(e, "email", 255, 8)}
          {handleKeyDown}
        />
      </div>
      <div class="mb-4">
        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="password"
          lable_name="Password"
          input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="password"
          placeholder="Enter password."
          input_value={register_form_data.password}
          isRequired="true"
          error={register_form_data_error.password}
          handleChange={(e) => handleInputValue(e, "password", 25, 8)}
          {handleKeyDown}
        />
      </div>
      <div class="mb-4">
        <Input
          lable_class="block text-gray-700 text-sm font-bold mb-2"
          input_name="confirm_password"
          lable_name="Confirm Password"
          input_class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          input_type="password"
          placeholder="Enter confirm password."
          input_value={register_form_data.confirm_password}
          isRequired="true"
          error={register_form_data_error.confirm_password}
          handleChange={(e) => handleInputValue(e, "password", 25, 8)}
          {handleKeyDown}
        />
      </div>
      <div class="flex items-center justify-between">
        <Button
          text="Sign Up"
          class_values="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
          type="submit"
        />
      </div>
      <div
        class="mt-3 flex items-center justify-between inline-block align-baseline text-sm font-bold"
      >
        Already have an account?
        <NavigateButton
          class_values="text-blue-500 hover:text-blue-800"
          text="Sign In"
          navigate_path={public_routes.login}
        />
      </div>
    </form>
  </div>
</PublicLayout>
