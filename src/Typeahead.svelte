<script>
  /**
   * @typedef {any} Item
   * @event {{ selectedIndex: number; selected: Item; }} select
   * @slot {{ result: { index: number; original: Item; score: number; string: string; } }}
   */

  export let id = "typeahead-" + Math.random().toString(36);
  export let value = "";

  /** @type {Item[]} */
  export let data = [];

  /** @type {(item: Item) => Item} */
  export let extract = (item) => item;
  export let autoselect = true;

  import fuzzy from "fuzzy";
  import Search from "svelte-search";
  import { tick, createEventDispatcher, afterUpdate } from "svelte";

  const dispatch = createEventDispatcher();

  let comboboxRef = undefined;
  let searchRef = undefined;
  let hideDropdown = false;
  let selectedIndex = -1;
  let prevResults = "";

  afterUpdate(() => {
    if (prevResults !== resultsId && autoselect) {
      selectedIndex = 0;
    }

    prevResults = resultsId;
  });

  async function select() {
    value = extract(results[selectedIndex].original);
    dispatch("select", { selectedIndex, selected: value });
    await tick();
    searchRef.focus();
    hideDropdown = true;
  }

  $: options = { pre: "<mark>", post: "</mark>", extract };
  $: results = fuzzy
    .filter(value, data, options)
    .filter(({ score }) => score > 0);
  $: resultsId = results.map((result) => extract(result.original)).join("");
</script>

<style>
  .display-block {
    display: block;
  }
  
  .svelte-typeahead {
    position: relative;
  }

  :global(.svelte-typeahead.dropdown .svelte-search input) {
    border-bottom-right-radius: 0;
    border-bottom-left-radius: 0;
  }

  :global(.svelte-search input) {
    border: 0;
    background: none;
    width: 100%;
    font: inherit;
    font-size: 1.5rem;
    padding: 1rem;
    border: 2px solid #e0e0e0;
    border-radius: 0.25rem;
  }

  :global(.svelte-search input:focus) {
    outline: 0;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    border-color: #0f62fe;
  }
</style>

<svelte:window
  on:click="{({ target }) => {
    if (!hideDropdown && results.length > 0 && comboboxRef && !comboboxRef.contains(target)) {
      hideDropdown = true;
    }
  }}"
/>

<div
  bind:this="{comboboxRef}"
  role="combobox"
  aria-haspopup="listbox"
  aria-owns="{id}-listbox"
  class="dropdown"
  
  aria-expanded="{!hideDropdown && results.length > 0}"
  id="{id}"
>
  <Search
    {...$$restProps}
    bind:this="{searchRef}"
    aria-autocomplete="list"
    aria-controls="{id}-listbox"
    aria-labelledby="{id}-label"
    aria-activedescendant=""
    id="{id}"
    bind:value
    on:input
    on:change
    on:focus
    on:focus="{() => {
      hideDropdown = false;
    }}"
    on:clear="{() => {
      hideDropdown = false;
    }}"
    on:blur
    on:keydown
    on:keydown="{({ key }) => {
      switch (key) {
        case 'Enter':
          select();
          break;
        case 'ArrowDown':
          selectedIndex += 1;
          if (selectedIndex === results.length) {
            selectedIndex = 0;
          }
          break;
        case 'ArrowUp':
          selectedIndex -= 1;
          if (selectedIndex < 0) {
            selectedIndex = results.length - 1;
          }
          break;
        case 'Escape':
          value = '';
          searchRef.focus();
          hideDropdown = true;
          break;
      }
    }}"
  />

  <div class="dropdown-menu" class:display-block="{!hideDropdown}" id="{id}-listbox" role="menu">
    {#if !hideDropdown && results.length > 0}
        <div class="dropdown-content">
          {#each results as result, i}
            <a id="{id}-result"
               class="dropdown-item"
               class:is-active="{selectedIndex === i}"
               aria-selected="{selectedIndex === i}"
               on:click="{() => {
                 selectedIndex = i;
                 select();
               }}"
              >
                {@html result.string}
            </a>
          {/each}
        </div>
    {/if}
  </div>
</div>
