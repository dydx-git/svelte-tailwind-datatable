<script>
  import Tailwindcss from "./Tailwindcss.svelte";
  import Datatable from "./datatable/DataTable.svelte";
  import data from "./datatable/data/data.js";
  import Form from "./Form.svelte";
  import { slide } from "svelte/transition";

  let showEditForm = false;

  const handleRowClick = e => {
    if (e.detail.target.name) showEditForm = true;
  };

  let columndata = [
    // Array of objects
    {
      label: "First Name", // Column name
      field: "name", // Field name from row
      numeric: false, // Affects sorting
      html: false // Escapes output if false.
    },
    {
      label: "Last Name",
      field: "price",
      numeric: false,
      html: false
    },
    {
      label: "Age",
      field: "clientid",
      numeric: true,
      html: false
    },
    {
      label: "State",
      field: "vendorid",
      numeric: false,
      html: false
    },
    {
      label: "State",
      field: "date",
      numeric: false,
      html: false
    },
    {
      label: "Action",
      field: function(data) {
        return (
          '<div class="py-2">' +
          '<Button class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold py-2 px-4 rounded-l" name="edit_button">Show dialog</Button>' +
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
  <div transition:slide>
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
    <Form />
  </div>
{/if}
