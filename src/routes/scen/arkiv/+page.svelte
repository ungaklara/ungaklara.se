<script>
  import { asText } from '@prismicio/client'
  import { page } from '$app/stores'
  import { onMount } from 'svelte'

  import GridCell from '$lib/GridCell.svelte'
  import RichText from '$lib/RichText.svelte'
  import { track } from '$lib/utils/track.js'
  import { resolve } from '$lib/prismic.js'
  import Grid from '$lib/Grid.svelte'
  import Html from '$lib/Html.svelte'
  import Card from '$lib/Card.svelte'
  import Scen from '../+page.svelte'
  import { asImageWidthSrcSet } from '@prismicio/helpers'

  export let data

  $: tag = $page.url.searchParams.get('tag')
  $: period = $page.url.searchParams.get('period')

  const onclick = (event) => () => {
    track('select_item', {
      item_list_name: 'Aktuellt',
      items: [
        {
          item_id: event.id,
          item_name: asText(event.data.title),
          item_category: 'Produktion'
        }
      ]
    })
  }

  onMount(function () {
    track('view_item_list', {
      item_list_name: 'Arkiv',
      items: data.events.map((event) => ({
        item_id: event.id,
        item_name: asText(event.data.title),
        item_category: 'Produktion'
      }))
    })
  })

  function image(props) {
    if (!props.url) return null
    const sources = asImageWidthSrcSet(props, {widths: [400, 600, 900], ar: '1:1'})
    return {
      srcset: sources?.srcset,
      sizes: '(min-width: 600px) 33vw, 100vw',
      alt: props.alt || '',
      src: sources?.src,
      width: props.dimensions.width,
      height: props.dimensions.width
    }
  }
</script>

<Scen {data} {period} {tag} tab="arkiv">
  <Grid ordered appear size={{ xs: '1of2', md: '1of3', lg: '1of4' }}>
    {#each data.events as event}
      <GridCell>
        <Card
          shrink
          clamp={4}
          on:click={onclick(event)}
          title={asText(event.data.title)}
          image={image(event.data.poster)}
          link={{ href: resolve(event), text: event.data.cta || 'Läs mer' }}>
          <Html>
            <RichText content={event.data.description} />
          </Html>
        </Card>
      </GridCell>
    {/each}
  </Grid>
</Scen>
