<script>
  import { onMount } from 'svelte'

  let reachdeckTimer = null
  let reachdeckTimout = 3000
  let reachDeckContainerElement = null

  $: isReachdeckLoaded = false
  $: isReachdeckVisible = false

  function checkReachdeckLoaded() {
    const reachdeckWasLoaded = typeof window?.ReachDeck !== 'undefined'

    if (reachdeckTimout <= 0) {
      clearTimeout(reachdeckTimer)

      return
    }

    if (reachdeckTimout <= 0 || reachdeckWasLoaded) {
      clearTimeout(reachdeckTimer)

      isReachdeckLoaded = true

      return
    }

    reachdeckTimout -= 100

    reachdeckTimer = setTimeout(checkReachdeckLoaded, 100)
  }

  onMount(() => {
    const layoutElement = document.querySelector('body')

    reachdeckTimer = setTimeout(checkReachdeckLoaded, 100)

    const reachdeckAttributeObserver = new MutationObserver(function (
      mutations
    ) {
      mutations.forEach(function ({ type, attributeName, target }) {
        if (type === 'attributes' && attributeName === 'style') {
          isReachdeckVisible = !target?.getAttribute('style').includes('none')
        }
      })
    })

    const reachdeckInjectedObserver = new MutationObserver(function (
      mutations_list
    ) {
      mutations_list.forEach(function (mutation) {
        mutation.addedNodes.forEach(function (added_node) {
          if (added_node?.id === '__bs_entryDiv') {
            reachDeckContainerElement = added_node

            isReachdeckVisible = !added_node
              ?.getAttribute('style')
              .includes('none')

            reachdeckAttributeObserver.observe(reachDeckContainerElement, {
              attributes: true //configure it to listen to attribute changes
            })

            reachdeckInjectedObserver.disconnect()
          }
        })
      })
    })

    if (layoutElement) {
      reachdeckInjectedObserver.observe(layoutElement, {
        subtree: false,
        childList: true
      })
    }
  })
</script>

{#if isReachdeckLoaded}
  <div
    class="top-banner"
    aria-label="ReachDeck togglebar"
    data-reachdeck-visible={isReachdeckVisible}>
    <svelte:element
      this="a"
      id="bapluslogo"
      class="logo"
      onclick="if(typeof window.ReachDeck !== 'undefined') window.ReachDeck.panel.toggleBar();"
      href="#">
      Aktivera Talande Webb
    </svelte:element>
  </div>
{/if}

<style>
  .top-banner {
    height: 40px;
    display: none;
    padding-inline: 1rem;
    background-color: #000;
    color: #fff;
  }

  .top-banner a#bapluslogo {
    display: flex;
    place-items: center;
    place-content: center;
    width: 100%;
    height: 100%;
  }

  .top-banner a#bapluslogo:hover,
  .top-banner a#bapluslogo:focus-visible {
    text-decoration: underline;
  }

  .top-banner:not([data-reachdeck-visible='true']) {
    display: block;
  }
</style>
