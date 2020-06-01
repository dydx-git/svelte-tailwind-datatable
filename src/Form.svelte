<script>
  import Select from "svelte-select";
  import Checkbox from "svelte-checkbox";

  export let submit = () => console.log("Add button was clicked");
  export let cancel = () => console.log("Cancel button was clicked");

  export let rush = false;

  let handleSelectClient = selectedVal => console.log(selectedClient);

  let clients = [
    { value: "chocolate", label: "🍫 Chocolate", group: "Sweet" },
    { value: "pizza", label: "🍕 Pizza", group: "Savory" },
    { value: "cake", label: "🎂 Cake", group: "Sweet" },
    { value: "cookies", label: "🍪 Cookies", group: "Savory" },
    { value: "ice-cream", label: "🍦 Ice Cream", group: "Sweet" }
  ];

  export let selectedClient = 3;

  let vendors = [
    { value: "chocolate", label: "🍫 Chocolate", group: "Sweet" },
    { value: "pizza", label: "🍕 Pizza", group: "Savory" },
    { value: "cake", label: "🎂 Cake", group: "Sweet" },
    { value: "cookies", label: "🍪 Cookies", group: "Savory" },
    { value: "ice-cream", label: "🍦 Ice Cream", group: "Sweet" }
  ];

  export let selectedVendor = undefined;

  export let orderName = "";
  export let price = "";
</script>

<form class="container mx-auto my-64 w-full max-w-lg">
  <div class="flex flex-wrap -mx-3 mb-6">
    <div class="w-full md:w-1/2 px-3 mb-6 md:mb-0">
      <input
        bind:value="{orderName}"
        class="appearance-none block w-full text-gray-700 border
        border-black-400 rounded py-3 px-4 leading-tight focus:outline-none
        focus:bg-white focus:border-gray-500"
        type="text"
        placeholder="Order Name"
      />
      {#if orderName.length == 0}
        <p class="text-red-500 text-xs italic">Please fill out this field.</p>
      {/if}
    </div>
    <div class="w-full md:w-1/2 px-3">
      <input
        bind:value="{price}"
        class="appearance-none block w-32 text-gray-700 border border-black-400
        rounded py-3 px-4 leading-tight focus:outline-none focus:bg-white
        focus:border-gray-500"
        type="number"
        placeholder="Price"
      />
    </div>
  </div>
  <div class="flex flex-wrap -mx-3 mb-6">
    <div class="w-full md:w-1/2 px-3 mb-6 md:mb-0">
      <Select
        class="appearance-none block w-full text-gray-700 rounded py-3 px-4 mb-3
        leading-tight focus:outline-none focus:bg-white"
        items="{clients}"
        bind:selectedValue="{selectedClient}"
        on:select="{handleSelectClient}"
        placeholder="Select Client"
      />
    </div>
  </div>
  <div class="flex flex-wrap -mx-3 mb-2">
    <div class="w-full md:w-1/2 px-3 mb-6 md:mb-0">
      <Select
        class="appearance-none block w-full text-gray-700 rounded py-3 px-4 mb-3
        leading-tight focus:outline-none focus:bg-white"
        items="{vendors}"
        bind:selectedValue="{selectedVendor}"
        placeholder="Select Vendor"
      />
    </div>
    <div class="w-full md:w-1/2 px-3 mb-6 md:mb-0">
      <div class="flex mb-2">
        <div class="w-1/12> my-1">
          <Checkbox size="2rem" bind:checked="{rush}" />
        </div>
        <div class="w-1/12 my-1 mx-2 text-xl">
          <label>Rush?</label>
        </div>
      </div>
    </div>
  </div>
  <div
    class="px-5 py-5 bg-white border-t flex flex-col xs:flex-row items-center
    xs:justify-between "
  >
    <div class="mt-2 px-2 xs:mt-0 items-center">
      <button
        class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold
        py-2 px-4 rounded-l"
        on:click="{submit}"
        type="button"
      >
        Add
      </button>
      <button
        class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold
        py-2 px-4 rounded-r"
        on:click="{cancel}"
        type="button"
      >
        Cancel
      </button>
    </div>
  </div>

</form>
