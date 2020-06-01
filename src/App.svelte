<script>
  import Tailwindcss from "./Tailwindcss.svelte";
  import Datatable from "./datatable/DataTable.svelte";
  import data from "./datatable/data/data.js";
  import Form from "./Form.svelte";
  import { slide } from "svelte/transition";

  let showEditForm = false;

  let orderName;
  let price;
  let selectedClient;
  let selectedVendor;

  let cancel = () => (showEditForm = false);

  const handleRowClick = e => {
    if (e.detail.target.name === "editOrderBtn") {
      orderName = e.detail.row.name;
      price = e.detail.row.price;
      selectedClient = {
        value: "chocolate",
        label: "🍫 Chocolate",
        group: "Sweet"
      };
      selectedVendor = { value: "cake", label: "🎂 Cake", group: "Sweet" };
      showEditForm = true;
    }
  };

  let columndata = [
    // Array of objects
    {
      label: "First Name", // Column name
      field: "fname", // Field name from row
      numeric: false, // Affects sorting
      html: false // Escapes output if false.
    },
    {
      label: "Last Name",
      field: "lname",
      numeric: false,
      html: false
    },
    {
      label: "Age",
      field: "age",
      numeric: true,
      html: false
    },
    {
      label: "State",
      field: "state",
      numeric: false,
      html: false
    },
    {
      label: "Action",
      field: function(data) {
        return (
          '<div class="py-2">' +
          '<Button name="editOrderBtn" class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded" name="edit_button">Show dialog</Button>' +
          "</div>"
        );
      },
      numeric: false,
      html: true
    }
  ];
</script>

<Tailwindcss />

{#if !showEditForm}
  <div transition:slide="{{ duration: 100 }}">
    <Datatable
      rows="{data}"
      title="Orders"
      columns="{columndata}"
      process="local"
      on:row-click="{handleRowClick}"
    />
  </div>
{:else}
  <div>
    <Form {cancel} {orderName} {price} {selectedClient} {selectedVendor} />
  </div>
{/if}
