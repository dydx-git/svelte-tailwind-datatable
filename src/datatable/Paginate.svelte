<script>
  const splitObject = o => Object.keys(o).map(e => o[e]);

  export let selected = 0;
  export let pageCount = 0;
  export let pageRange = 3;
  export let marginPages = 1;
  export let containerClass = "";
  export let prevClass = "";
  export let disabledClass = "disabled";
  export let nextClass = "";
  export let clickHandler = () => {};

  export let pages;

  $: pages = getPages(pageCount, selected);

  function getPages(pageCount, selected) {
    let items = {};
    if (pageCount <= pageRange) {
      for (let index = 0; index < pageCount; index++) {
        let page = {
          index: index,
          content: index + 1,
          selected: index === selected
        };
        items[index] = page;
      }
    } else {
      let leftPart = pageRange / 2;
      let rightPart = pageRange - leftPart;

      if (selected < leftPart) {
        leftPart = selected;
        rightPart = pageRange - leftPart;
      } else if (selected > pageCount - pageRange / 2) {
        rightPart = pageCount - selected;
        leftPart = pageRange - rightPart;
      }

      // items logic extracted into this function
      let mapItems = index => {
        let page = {
          index: index,
          content: index + 1,
          selected: index === selected
        };

        if (index <= marginPages - 1 || index >= pageCount - marginPages) {
          items[index] = page;
          return;
        }

        let breakView = {
          content: "...",
          disabled: true
        };

        if (
          selected - leftPart > marginPages &&
          items[marginPages] !== breakView
        ) {
          items[marginPages] = breakView;
        }

        if (
          selected + rightPart < pageCount - marginPages - 1 &&
          items[pageCount - marginPages - 1] !== breakView
        ) {
          items[pageCount - marginPages - 1] = breakView;
        }

        let overCount = selected + rightPart - pageCount + 1;

        if (overCount > 0 && index === selected - leftPart - overCount) {
          items[index] = page;
        }

        if (index >= selected - leftPart && index <= selected + rightPart) {
          items[index] = page;
          return;
        }
      };

      // 1st - loop thru low end of margin pages
      for (let i = 0; i < marginPages; i++) {
        mapItems(i);
      }

      // 2nd - loop thru high end of margin pages
      for (let i = pageCount - 1; i >= pageCount - marginPages; i--) {
        mapItems(i);
      }

      // 3rd - loop thru selected range
      let selectedRangeLow = 0;
      if (selected - pageRange > 0) {
        selectedRangeLow = selected - pageRange;
      }

      let selectedRangeHigh = pageCount;
      if (selected + pageRange < pageCount) {
        selectedRangeHigh = selected + pageRange;
      }

      for (let i = selectedRangeLow; i < selectedRangeHigh; i++) {
        mapItems(i);
      }
    }
    return splitObject(items);
  }

  // [svelte-upgrade suggestion]
  // review these functions and remove unnecessary 'export' keywords
  export function handlePageSelected(e, selectedIndex) {
    e.preventDefault();
    if (selected === selectedIndex) return;

    selected = selected = selectedIndex;
    clickHandler(selected + 1);
  }

  export function prevPage(e) {
    e.preventDefault();
    if (selected <= 0) return;

    selected = --selected;
    clickHandler(selected + 1);
  }

  export function nextPage(e) {
    e.preventDefault();
    if (selected >= pageCount - 1) return;

    selected = ++selected;
    clickHandler(selected + 1);
  }

  function lastPageSelected(selected, pageCount) {
    return selected === pageCount - 1 || pageCount === 0;
  }

  function firstPageSelected(selected) {
    return selected === 0;
  }
</script>

<style>
  a {
    cursor: pointer;
  }
</style>

<div class={containerClass}>
  <button
    class="{prevClass}
    {firstPageSelected(selected) ? disabledClass : ''}"
    on:click={prevPage}>
    Prev
  </button>
  <button
    class="{nextClass}
    {lastPageSelected(selected, pageCount) ? disabledClass : ''}"
    on:click={nextPage}>
    Next
  </button>
</div>
