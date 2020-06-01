<script>
  import Datatable from "./datatable/DataTable.svelte";
  import data from "./datatable/data/data.js";

  const handleRowClick = e => {
    if (e.detail.target.name) console.log(e.detail.row);
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
          '<Button name="edit_button">Show dialog</Button>' +
          "</div>"
        );
      },
      numeric: false,
      html: true
    }
  ];
</script>

<Datatable
  rows="{data}"
  title="Orders"
  columns="{columndata}"
  process="local"
  on:row-click="{handleRowClick}"
/>
