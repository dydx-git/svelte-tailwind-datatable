<script>
  import { beforeUpdate, createEventDispatcher, onMount } from "svelte";
  import Fuse from "fuse.js";

  import Paginate from "./Paginate.svelte";
  import { debounce } from "./debounce";
  import * as local from "./local";
  import { collect, exportExcel, print } from "./data-grid";

  import { immerStore } from "./stores";

  export let title = "";
  export let customButtons = [];
  export let clickable = true;
  export let printable = true;
  export let exportable = true;
  export let searchable = true;
  export let searching = false;
  export let searchInputRef = null;
  export let searchInput = "";

  export let sortable = true;
  export let sortColumn = -1;
  export let sortIndex = -1;
  export let sortType = "asc";

  export let currentPerPage = 10;
  export let perPageOptions = [];
  export let currentPage = 1;
  export let rowCount = 0;
  export let pageCount = 0;
  export let selected = 0;
  export let perPage = [10, 20, 30, 40, 50];
  export let paginate = { size: perPage[0], page: 1 };
  export let defaultPerPage = null;
  export let exactSearch = false;

  export let columns = [];
  export let process = "server";
  // local
  export let rows = [];
  export let paginatedRows = [];
  // server
  export let paged = { paginate: paginate };
  export let getList = null;

  const dispatch = createEventDispatcher();
  const isServerProcess = process === "server";

  const paginated = immerStore({
    paginatedRows,
    rows,
    paginate,
    size: currentPerPage,
    page: currentPage
  });

  export const handleKeydown = e => {
    if (e.key === "/" && e.altKey) {
      searchInputRef.focus();
      searchInput = "";
    }
  };

  export function setSortIcon(index) {
    if (!sortable) return;
    if (sortColumn === index) {
      sortType = sortType === "asc" ? "desc" : "asc";
    } else {
      sortType = "asc";
      sortColumn = index;
    }
    (sortType = sortType), (sortColumn = sortColumn);
  }

  export function setPerPageOptions() {
    let options = perPage;
    // Force numbers
    options = options.map(v => parseInt(v));
    // Set current page to first value
    currentPerPage = options[0];
    // Sort options
    options.sort((a, b) => a - b);
    // And add "All"
    options.push(-1);
    // If defaultPerPage is provided and it's a valid option, set as current per page
    if (options.indexOf(defaultPerPage) > -1) {
      currentPerPage = parseInt(defaultPerPage);
    }
    console.log("currentPerPage", currentPerPage, options);
    (currentPerPage = currentPerPage), (perPageOptions = options);
  }

  export function click(row) {
    if (!clickable) {
      return;
    }
    if (getSelection().toString()) {
      // Return if some text is selected instead of firing the row-click event.
      return;
    }
    dispatch("row-click", row);
  }

  export function sort(index) {
    if (index > -1) {
      setSortIcon(index);
      getPaged({ colName: columns[index].field, direction: sortType });
    }
  }

  onMount(() => {
    setPerPageOptions();
    getPaged({ rows });
  });

  function getPagedLocal(props, pred) {
    const values = local.getPaged(props, pred, {
      paginate,
      selected,
      currentPage,
      currentPerPage,
      sortable,
      sortColumn,
      sortType,
      searchInput,
      exactSearch,
      rowCount,
      pageCount,
      rows,
      columns
    });
    ({
      paginate,
      selected,
      currentPage,
      currentPerPage,
      sortable,
      sortColumn,
      sortType,
      searchInput,
      exactSearch,
      rowCount,
      pageCount,
      rows,
      paginatedRows
    } = values);
    update(paginatedRows, currentPerPage, selectedPage);
  }

  function getPagedServer(props, pred, paged, cb) {
    const { paginate, getList } = paged;
    if (getList && (!pred || pred(paginate))) {
      const p = Object.assign({}, paginate, props);
      getList(p).then(data => {
        console.log("getPagedServer ", data);
        cb(data);
      });
    }
  }

  function updateServer(paged) {
    const { paginate } = paged;
    rowCount = paginate.total;
    pageCount = paginate.pages;
    currentPage = paginate.page;
    if (paged.getList) {
      getList = paged.getList;
    }
  }

  function updatePaged(data) {
    updateServer(data);
    update(data.rows, data.paginate.size, data.paginate.page);
  }

  function getPaged(props, pred) {
    console.log("props", props);
    if (isServerProcess) {
      getPagedServer(props, pred, { paginate, getList }, updatePaged);
    } else {
      getPagedLocal(props, pred);
    }
  }

  $: selectedPage = selected + 1;
  let previous;
  function update(paginatedRows, size, page) {
    paginated.update($paginated => {
      $paginated.paginatedRows = paginatedRows;
      $paginated.rows = rows;
      $paginated.size = size;
      $paginated.page = page;
    });
    previous = $paginated;
  }

  $: {
    // console.log('$paginated - previous - selectedPage ', $paginated.page, previous ? previous.page : -1, selectedPage);
    if (previous) {
      if (!isServerProcess && rows !== previous.rows) {
        getPaged({ rows });
      }
      if (currentPerPage !== previous.size) {
        getPaged({ size: currentPerPage });
      }
      if (selectedPage !== previous.page) {
        getPaged({ page: selectedPage }, x => x.page != selectedPage);
        paginate.page = selectedPage;
      }
    }
    previous = $paginated;
  }

  let prevPaged;
  $: if (paged.rows && paged !== prevPaged) {
    updatePaged(paged);
    prevPaged = paged;
    console.log("paged !== prevPaged ", $paginated);
  }

  let prevSearchInput = "";
  beforeUpdate(() => {
    if (searchInput != prevSearchInput) {
      debounce(() => {
        if (searchInput != prevSearchInput) {
          console.log("searchInput changed", searchInput);
          getPaged({ searchText: searchInput });
          prevSearchInput = searchInput;
        }
      }, 400)();
    }
  });
</script>

<!-- component starts here -->
<div class="container mx-auto px-4 sm:px-8">
  <div class="py-8">
    <div>
      <h2 class="text-2xl font-semibold leading-tight">{title}</h2>
    </div>
    <div class="my-2 flex sm:flex-row flex-col">
      <div class="flex flex-row mb-1 sm:mb-0">
        <div class="relative">
          <select
            class="appearance-none h-10 rounded-l border-none block
            appearance-none w-full bg-white border-gray-400 text-gray-700 py-2
            px-4 pr-8 leading-tight focus:outline-none focus:bg-white
            focus:border-gray-500"
            bind:value="{currentPerPage}"
          >
            {#each perPageOptions as option, x}
              <option value="{option}">{option === -1 ? 'All' : option}</option>
            {/each}
          </select>
        </div>
      </div>
      <div class="block relative">
        <span class="h-full absolute inset-y-0 left-0 flex items-center pl-2">
          <svg viewBox="0 0 24 24" class="h-4 w-4 fill-current text-gray-500">
            <path
              d="M10 4a6 6 0 100 12 6 6 0 000-12zm-8 6a8 8 0 1114.32 4.906l5.387
              5.387a1 1 0 01-1.414 1.414l-5.387-5.387A8 8 0 012 10z"
            ></path>
          </svg>
        </span>
        <input
          placeholder="Search"
          type="search"
          id="search-input"
          bind:this="{searchInputRef}"
          bind:value="{searchInput}"
          class="appearance-none rounded-r rounded-l sm:rounded-l-none border
          border-gray-400 border-dashed block pl-8 pr-6 py-2 h-10 w-full
          bg-white text-sm placeholder-gray-400 text-gray-700 focus:bg-white
          focus:placeholder-gray-600 focus:text-gray-700 focus:outline-none"
        />
      </div>
      <div class="flex ml-auto">
        <button
          class="text-sm mr-6 bg-gray-300 hover:bg-gray-400 text-gray-800
          font-semibold py-2 px-4 rounded"
        >
          Add new design
        </button>
      </div>
    </div>

    <div class="-mx-4 sm:-mx-8 px-4 sm:px-8 py-4 overflow-x-auto">
      <div class="inline-block min-w-full shadow rounded-lg overflow-hidden">
        <table class="min-w-full leading-normal">
          <thead>
            <tr>
              {#each columns as column, x}
                <th
                  on:click="{() => sort(x)}"
                  class="{sortable ? 'sorting ' : ''}
                  {sortColumn === x ? (sortType === 'desc' ? 'sorting-desc' : 'sorting-asc') : ''}
                  {column.numeric ? ' numeric' : ''} px-5 py-3 border-b-2
                  border-gray-200 bg-gray-100 text-left text-xs font-semibold
                  text-gray-600 uppercase tracking-wider"
                  style="width: {column.width ? column.width : 'auto'}"
                >
                  {column.label}
                </th>
              {/each}
            </tr>
          </thead>

          <tbody>
            {#each $paginated.paginatedRows as row, y}
              <tr
                class="{clickable ? 'clickable' : ''}"
                on:click="{e => click({ target: e.target, row })}"
              >
                {#each columns as column, x}
                  <td
                    class="px-2 py-1 border-b border-gray-200 bg-white text-sm"
                  >
                    <p class="text-gray-900 whitespace-no-wrap">
                      {#if column.html}
                        {@html collect(row, column.field)}
                      {:else}{collect(row, column.field)}{/if}
                    </p>
                  </td>
                {/each}
              </tr>
            {/each}
          </tbody>
        </table>
        <div
          class="px-5 py-5 bg-white border-t flex flex-col xs:flex-row
          items-center xs:justify-between "
        >
          <span class="text-xs xs:text-sm text-gray-900">
            Showing {(currentPage - 1) * currentPerPage ? (currentPage - 1) * currentPerPage : 1}
            to {Math.min(rowCount, currentPerPage * currentPage)} of {rowCount}
            entries
          </span>
          <div class="inline-flex mt-2 xs:mt-0">
            <Paginate
              bind:pageCount
              marginPages="2"
              pageRange="4"
              bind:selected
              containerClass="inline-flex mt-2 xs:mt-0"
              prevClass="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800
              font-semibold py-2 px-4 rounded-l"
              nextClass="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800
              font-semibold py-2 px-4 rounded-r"
            >
              <button
                class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800
                font-semibold py-2 px-4 rounded-l"
              >
                Prev
              </button>
              <button
                class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800
                font-semibold py-2 px-4 rounded-r"
              >
                Next
              </button>
            </Paginate>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
<!-- component ends here -->

<svelte:window on:keydown="{handleKeydown}" />

<!-- <div class="container">
  <div class="material-table">
    <div class="table-header">
      <span class="table-title">{title}</span>
      <div class="actions">
        {#each customButtons as button, x}
          {#if !button.hide}
            <a
              href="#!"
              class="waves-effect btn-flat nopadding"
              on:click={click}>
              <i class="material-icons">{button.icon}</i>
            </a>
          {/if}
        {/each}
        {#if printable}
          <a
            href="#!"
            class="waves-effect btn-flat nopadding"
            on:click={() => print(columns, rows)}>
            <i class="material-icons">print</i>
          </a>
        {/if}
        {#if exportable}
          <a
            href="#!"
            class="waves-effect btn-flat nopadding"
            v-if="this.exportable"
            on:click={() => exportExcel(columns, rows, title)}>
            <i class="material-icons">description</i>
          </a>
        {/if}
        {#if searchable}
          <a
            href="#!"
            class="waves-effect btn-flat nopadding"
            on:click={search}>
            <i class="material-icons">search</i>
          </a>
        {/if}
      </div>
    </div>
    {#if searching}
      <div>
        <div id="search-input-container">
          <label>
            <input
              type="search"
              id="search-input"
              bind:this={searchInputRef}
              class="form-control"
              placeholder="Search data"
              bind:value={searchInput} />
          </label>
        </div>
      </div>
    {/if}

    <table ref="table" class="table table-striped table-hover">
      <thead>
        <tr>
          {#each columns as column, x}
            <th
              on:click={() => sort(x)}
              class="{sortable ? 'sorting ' : ''}
              {sortColumn === x ? (sortType === 'desc' ? 'sorting-desc' : 'sorting-asc') : ''}
              {column.numeric ? ' numeric' : ''}"
              style="width: {column.width ? column.width : 'auto'}">
              {column.label}
            </th>
          {/each}
        </tr>
      </thead>

      <tbody>
        {#each $paginated.paginatedRows as row, y}
          <tr class="{ clickable ? 'clickable' : '' }" on:click="{(e) => click({target: e.target, row})}">
            {#each columns as column, x}
              <td>
                {#if column.html}
                  {@html collect(row, column.field)}
                {:else}{collect(row, column.field)}{/if}
              </td>
            {/each}
          </tr>
        {/each}
      </tbody>
    </table>
    {#if paginate}
      <div class="table-footer">
        <div class="datatable-length">
          <label>
            <span>Rows per page:</span>
            <select class="browser-default" bind:value={currentPerPage}>
              {#each perPageOptions as option, x}
                <option value={option}>{option === -1 ? 'All' : option}</option>
              {/each}
            </select>
          </label>
        </div>
        <div class="datatable-info">
          {(currentPage - 1) * currentPerPage ? (currentPage - 1) * currentPerPage : 1}
          -{Math.min(rowCount, currentPerPage * currentPage)} of {rowCount}
        </div>
        <div>
          <Paginate
            bind:pageCount
            marginPages="2"
            pageRange="4"
            containerClass="pagination material-pagination"
            pageLinkClass="waves-effect btn-flat"
            prevLinkClass="waves-effect btn-flat nopadding"
            nextLinkClass="waves-effect btn-flat nopadding"
            bind:selected
            activeClass="active">
            <i class="material-icons" slot="prevContent">chevron_left</i>
            <i class="material-icons" slot="nextContent">chevron_right</i>
          </Paginate>
        </div>
      </div>
    {/if}
  </div>
</div> -->

<style>

</style>
