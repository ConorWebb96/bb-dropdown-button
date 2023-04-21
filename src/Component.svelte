<script>
  import { getContext, onMount } from "svelte";

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  // schema exports
  export let buttonText;
  export let dPosition;
  export let width;
  export let size = "M";
  export let type;
  export let quiet;
  export let newStyles = true
  export let disabled = false


  // icon exports
  export let icon;
  export let iconColour;
  export let iconSize = "M";

  $: iconStyles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      color: iconColour || "var(--spectrum-global-color-white-900)",
    },
  }

  // preset vars to be used later
  let dropdownContentStyle = 'hidden';
  let randomString = "";

  onMount(() => {
      // generate random characters on mounted to asigned to buttons later.
      const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
      for (let i = 0; i < 10; i++) {
        randomString += characters.charAt(Math.floor(Math.random() * characters.length));
      }
  });
  const dropdownButtonClicked = () => {
      positionDropdown(); // Positions the dropdown relative to the button
      if (dropdownContentStyle === 'hidden') { // If the dropdown is currently hidden
        dropdownContentStyle = 'show'; // Show the dropdown
        document.addEventListener('click', handleClickAway); // Add a click event listener to detect clicks outside of the dropdown
      } else { // If the dropdown is currently shown
        dropdownContentStyle = 'hidden'; // Hide the dropdown
        document.removeEventListener('click', handleClickAway); // Remove the click event listener
      }
  };
  const handleClickAway = (event) => {
      const dropdownContainer = document.getElementById(`dropdown-container-${randomString}`); // Get the dropdown container element
      if (!dropdownContainer.contains(event.target)) { // If the clicked element is not a child of the dropdown container
        dropdownContentStyle = 'hidden'; // Hide the dropdown
        document.removeEventListener('click', handleClickAway); // Remove the click event listener
      }
  };
  const positionDropdown = () => {
      // button component variables
      const dropdownButton = document.getElementById(`dropdown-button-${randomString}`);
      const component = dropdownButton.closest('.component');
      const parentOfComponent = component.parentElement.parentElement; // set table cell
      const isTable = component.parentElement.parentElement.classList.contains('spectrum-Table-cell'); // check if the button is inside a table

      const dropdownContainer = document.getElementById(`dropdown-container-${randomString}`);
      const dropdownContent = document.getElementById(`dropdown-content-${randomString}`);
      // div const for rects
      const containerRect = dropdownContainer.getBoundingClientRect();
      const contentRect = dropdownContent.getBoundingClientRect();

      let tableRect = null;
      // get table rect if inside a table
      if (isTable) {
        const table = parentOfComponent.closest('.spectrum-Table');
        if (table) {
          tableRect = table.getBoundingClientRect();
        }
      }
      // spaceing consts
      let spaceAbove = containerRect.top - contentRect.height;
      let spaceBelow = window.innerHeight - containerRect.bottom - contentRect.height;
      let spaceLeft = containerRect.left;
      let spaceRight = window.innerWidth - containerRect.right;
      // adjust space constants if inside table
      if (tableRect) {
        spaceAbove = containerRect.top - (tableRect.top + contentRect.height);
        spaceBelow = tableRect.bottom - (containerRect.bottom + contentRect.height);
        spaceLeft = containerRect.left - tableRect.left;
        spaceRight = tableRect.right - containerRect.right;
      }

      dropdownContent.removeAttribute('style'); // reset style attr
      dropdownContent.style.minWidth = width + 'px'; // set width of dropdown

      switch (dPosition) {
        case 'left':
          dropdownContent.style.right = spaceLeft >= width || spaceLeft > spaceRight ? '100%' : 'auto';
          dropdownContent.style.left = spaceLeft >= width || spaceLeft > spaceRight ? 'auto' : '100%';
          dropdownContent.style.top = spaceBelow >= width || spaceBelow > spaceAbove ? '0%' : 'auto';
          dropdownContent.style.bottom = spaceBelow >= width || spaceBelow > spaceAbove ? 'auto' : '0%';
          break;
        case 'right':
          dropdownContent.style.left = spaceRight >= width || spaceRight > spaceLeft ? '100%' : 'auto';
          dropdownContent.style.right = spaceRight >= width || spaceRight > spaceLeft ? 'auto' : '100%';
          dropdownContent.style.top = spaceBelow >= width || spaceBelow > spaceAbove ? '0%' : 'auto';
          dropdownContent.style.bottom = spaceBelow >= width || spaceBelow > spaceAbove ? 'auto' : '0%';
          break;
        case 'bottom':
          dropdownContent.style.top = spaceBelow >= width || spaceBelow > spaceAbove ? '100%' : 'auto';
          dropdownContent.style.bottom = spaceBelow >= width || spaceBelow > spaceAbove ? 'auto' : '100%';
          dropdownContent.style.left = spaceRight >= width || spaceRight > spaceLeft ? '0%' : 'auto';
          dropdownContent.style.right = spaceRight >= width || spaceRight > spaceLeft ? 'auto' : '0%';
          break;
        default:
          dropdownContent.style.bottom = spaceAbove >= width || spaceAbove > spaceBelow ? '100%' : 'auto';
          dropdownContent.style.top = spaceAbove >= width || spaceAbove > spaceBelow ? 'auto' : '100%';
          dropdownContent.style.left = spaceRight >= width || spaceRight > spaceLeft ? '0%' : 'auto';
          dropdownContent.style.right = spaceRight >= width || spaceRight > spaceLeft ? 'auto' : '0%';
          break;
      }
  };
</script>

<!-- class:spectrum-Button--cta={cta}
class:spectrum-Button--primary={primary}
class:spectrum-Button--secondary={secondary}
class:spectrum-Button--warning={warning}
class:spectrum-Button--overBackground={overBackground} -->

<div id="dropdown-container-{randomString}" class="dropdown-container" use:styleable={$component.styles}>
    <button 
      id="dropdown-button-{randomString}" on:click={dropdownButtonClicked}
      class:spectrum-Button--quiet={quiet}
      class:new-styles={newStyles}
      class:disabled
      class="spectrum-Button spectrum-Button--size{size.toUpperCase()} spectrum-Button--{type}"
      {disabled}
    >
      <span>{buttonText}</span>
      {#if icon != undefined }
          <i
            use:styleable={iconStyles}
            class="{icon} {iconSize}"
          />
      {:else}
         <svg class="spectrum-Icon spectrum-UIIcon-ChevronDown100 {dropdownContentStyle}">
           <use xlink:href="#spectrum-css-icon-Chevron100" />
         </svg>
      {/if}
    </button>
    <div id="dropdown-content-{randomString}" class="dropdown-content spectrum-Popover is-open {dropdownContentStyle}">
      <slot />
    </div>
</div>

<style>
    .dropdown-container {
      position: relative;
      width: fit-content;
    }
    .dropdown-content {
      display: none;
      position: absolute;
      z-index: 25;
      padding: 1rem;
      overflow: scroll;
      max-width: 80vw;
      max-height: 80vh;
      -ms-overflow-style: none;  /* IE and Edge */
      scrollbar-width: none;  /* Firefox */
    }
    .dropdown-content::-webkit-scrollbar {
      display: none;
    }
    .dropdown-content.show {
      display: flex!important;
      flex-direction: column;
    }
    /* button styling */
    .spectrum-Button--primary.new-styles {
      background: var(--spectrum-global-color-gray-800);
      border-color: transparent;
      color: var(--spectrum-global-color-gray-50);
    }
    .spectrum-Button--primary.new-styles:hover {
      background: var(--spectrum-global-color-gray-900);
    }
    .spectrum-Button--secondary.new-styles {
      background: var(--spectrum-global-color-gray-200);
      border-color: transparent;
      color: var(--spectrum-global-color-gray-900);
    }
    .spectrum-Button--secondary.new-styles:not(.disabled):hover {
      background: var(--spectrum-global-color-gray-300);
    }
    .spectrum-Button--secondary.new-styles.disabled {
      color: var(--spectrum-global-color-gray-500);
    }
    .spectrum-Button {
      display: flex;
      align-items: center;
    }
    .spectrum-Button span {
      display: block;
      margin-right: 0.5rem;
    }
    .spectrum-Icon {
      transition: transform 0.5s ease-in-out;
    }
    .spectrum-Icon.show {
      transform: rotate(270deg);
    }
</style>