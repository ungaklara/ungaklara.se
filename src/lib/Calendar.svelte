<script>
  import { formatInTimeZone } from 'date-fns-tz'
  import parseJSON from 'date-fns/parseJSON/index.js'
  import { asText } from '@prismicio/client'
  import { onMount } from 'svelte'

  import { hexToRgb, luma } from '$lib/utils/colors.js'
  import { track } from '$lib/utils/track.js'
  import { resolve } from '$lib/prismic.js'
  import Button from '$lib/Button.svelte'
  import Framed from '$lib/Framed.svelte'
  import Symbol from '$lib/Symbol.svelte'
  import { asImageWidthSrcSet } from '@prismicio/helpers'

  export let events = []
  export let compact = false

  /** @type {number?} */
  export let limit = null

  const sort = (days) => {
    Object.values(days).forEach((items) => {
      items.sort((a, b) =>
        parseJSON(a.show.start) < parseJSON(b.show.start) ? -1 : 1
      )
    })
    return days
  }

  $: days = sort(
    events
      .flatMap((event) => {
        return event.production?.shows?.map((show) => {
          const [date] = show.start.split('T')
          return { event, show, date }
        })
      })
      .filter(Boolean)
      .sort((a, b) => (a.date < b.date ? -1 : 1))
      .reduce((acc, item) => {
        if (limit && Object.values(acc).flat().length >= limit) return acc
        if (acc[item.date]) acc[item.date].push(item)
        else acc[item.date] = [item]
        return acc
      }, {})
  )

  const onclick = (event, show) => () => {
    track('select_item', {
      item_list_name: 'Kalendarium',
      items: [showAsItem(show, event)]
    })
  }

  onMount(function () {
    track('view_item_list', {
      item_list_name: 'Kalendarium',
      items: events.flatMap((event) =>
        event.production?.shows.map((show) => showAsItem(show, event))
      )
    })
  })

  function showAsItem(show, event) {
    return {
      item_id: show.id,
      item_name: show.name,
      item_category: 'Föreställning',
      item_category2: asText(event.data.title),
      item_category3: new Date(show.start).toLocaleDateString('sv')
    }
  }

  function image(props) {
    if (!props.url) return null
    const sources = asImageWidthSrcSet(props, {widths: [64, 96, 128, 192]})
    return {
      srcset: sources?.srcset,
      sizes: compact ? '4rem' : '(min-width: 600px) 6rem, 4rem',
      alt: props.alt || '',
      src: sources?.src,
      ...props.dimensions
    }
  }
</script>

<ol class="calendar" class:compact>
  {#each Object.entries(days) as [key, items], index (key)}
    {@const [year, month, day] = items.at(0).date.split('-')}
    {@const date = new Date(+year, +month - 1, +day)}
    <li class="row" class:u-slideUp={!compact} style:--delay="{index * 100}ms">
      <time datetime="{year}-{month}-{day}" class="day">
        {date.toLocaleString('sv', {
          weekday: 'long'
        })}, {date.toLocaleString('sv', {
          year: 'numeric',
          day: 'numeric',
          month: 'long'
        })}
      </time>
      {#each items as item}
        {@const { event, show } = item}
        {@const start = parseJSON(show.start)}
        {@const color = event.data.theme ? hexToRgb(event.data.theme) : null}
        {@const isSoldOut = show.stockStatus === 'SoldOut'}
        <div
          class="show"
          style:--theme-color={color}
          style:--text-color={color && luma(color) < 110
            ? '255, 255, 255'
            : null}>
          {#if event.data.poster.url}
            <div class="poster">
              <Framed size="small" {...image(event.data.poster)} />
            </div>
          {/if}
          <div class="body">
            <div class="description">
              <a href={resolve(event)} class="link">
                {show.name}
              </a>
              <div class="meta">
                {#if !compact}
                  <span class="location">
                    <span class="icon"><Symbol name="location" /></span>
                    {show.venue.name}
                  </span>
                {/if}
                <span class="time">
                  <span class="icon"><Symbol name="clock" /></span>
                  {formatInTimeZone(start, 'Europe/Stockholm', 'HH:mm')}
                </span>
                {#if isSoldOut}
                  <span class="note">
                    {#if isSoldOut}Slutsåld{/if}
                  </span>
                {/if}
              </div>
            </div>
            <div class="actions {isSoldOut ? 'unavailable' : ''}">
              {#if show.stockStatus !== 'SoldOut'}
                <Button
                  primary
                  target="_blank"
                  disabled={isSoldOut}
                  rel="noopener noreferrer"
                  href={show.shopUri}
                  size={compact ? 'small' : null}
                  on:click={isSoldOut ? null : onclick(event, show)}>
                  {#if compact}Boka{:else}Boka biljett{/if}
                  {#if show.stockStatus === 'FewLeft'}
                    <span class="tag">
                      {#if show.stockStatus === 'FewLeft'}Få kvar!{/if}
                    </span>
                  {/if}
                </Button>
              {/if}
              {#if isSoldOut}
                <span class="note">
                  {#if isSoldOut}Slutsåld{/if}
                </span>
              {/if}
            </div>
          </div>
        </div>
      {/each}
    </li>
  {/each}
</ol>

<style>
  .calendar {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  @media (min-width: 800px) {
    .compact {
      display: block;
      column-count: 2;
      column-gap: 2rem;
    }
  }

  .row {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    position: relative;
  }

  @media (min-width: 600px) {
    .row {
      align-items: flex-start;
    }
  }

  @media (min-width: 800px) {
    .compact .row {
      break-inside: avoid-column;
      margin-bottom: 1.5rem;
    }
  }

  .day {
    display: block;
    width: 100%;
    border-top: var(--border-width) solid;
    padding: 0.5rem 0;
    font-family: var(--heading-font-family);
    line-height: var(--heading-line-height);
    text-transform: capitalize;
    font-size: 1rem;
  }

  @media (min-width: 800px) {
    .calendar:not(.compact) .day {
      font-size: 1.25rem;
      margin-top: 2rem;
    }
  }

  .show {
    display: flex;
    width: 100%;
    position: relative;
    z-index: 0;
  }

  .show + .show {
    border-top: 2px solid #d2d4da;
    padding-top: 1.15rem;
    margin-top: 0.75rem;
  }

  .poster {
    width: 4rem;
    margin-top: 0.25rem;
    flex-shrink: 0;
  }

  @media (min-width: 600px) {
    .calendar:not(.compact) .poster {
      width: 6rem;
    }
  }

  .body {
    flex: 1 1 auto;
    font-size: 1rem;
    margin-left: 1rem;
  }

  .compact .body {
    font-size: 1.125rem;
    display: flex;
    margin-top: 0;
    margin-left: 1.5rem;
    gap: 1rem;
    align-items: flex-start;
    justify-content: space-between;
  }

  @media (min-width: 600px) {
    .body {
      font-size: 1.125rem;
      display: flex;
      margin-top: 0;
      margin-left: 1.5rem;
      align-items: flex-start;
      justify-content: space-between;
    }
  }

  .description {
    flex: 1 1 auto;
  }

  .actions {
    margin-top: 0.5rem;
    display: flex;
    align-items: baseline;
    white-space: nowrap;
    flex-wrap: wrap;
    position: relative;
  }

  .actions:not(.unavailable) {
    z-index: 1;
  }

  @media (min-width: 600px) {
    .actions {
      display: block;
      margin-left: 2rem;
      margin-top: 1.25rem;
    }
  }

  .link {
    display: block;
    padding-bottom: 0.05rem;
    text-decoration: none;
    font-size: 1.5rem;
    font-family: var(--heading-font-family);
    line-height: 1.07;
    margin-top: 0.25rem;
    letter-spacing: var(--heading-letter-spacing);
    word-spacing: var(--heading-word-spacing);
    text-wrap: balance;
  }

  @media (min-width: 600px) {
    .link {
      font-size: 1.5rem;
      margin-top: 1rem;
    }
  }

  @media (min-width: 800px) {
    .link {
      font-size: 1.875rem;
    }
  }

  .link::after {
    content: '';
    position: absolute;
    inset: 0;
    z-index: 1;
  }

  .compact .link {
    margin: 0;
    font-size: 1.2rem;
  }

  .meta {
    margin-top: 0.25rem;
    display: flex;
    flex-wrap: wrap;
    margin-bottom: 0;
  }

  .icon {
    font-size: 0.8em;
    position: relative;
  }

  .time,
  .location,
  .note {
    display: inline-block;
    margin-right: 1rem;
    display: flex;
    align-items: baseline;
  }

  .compact .time,
  .compact .location,
  .compact .note {
    font-size: 0.9375rem;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .note {
    display: none;
  }

  .actions .note {
    display: block;
    margin-right: 0;
    text-align: right;
    margin-top: 0.25rem;
    min-height: 2.5rem;
    padding: 0 1rem;
    background: #d2d4d9;
    border-radius: var(--border-radius);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .compact .actions .note {
    padding: 0;
    min-width: 4.25rem;
  }

  .actions .note:first-child {
    margin-top: 0;
  }

  .location {
    display: none;
  }

  @media (min-width: 1000px) {
    .location {
      display: block;
    }
  }

  .link::before {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    z-index: -1;
  }

  .link:hover {
    text-decoration: underline;
    text-underline-offset: 0.15em;
    text-decoration-thickness: var(--border-width);
  }

  .time {
    display: block;
  }

  .tag {
    background: rgb(var(--theme-color));
    display: block;
    border-radius: var(--border-radius);
    width: 4.2rem;
    height: 1.55rem;
    white-space: normal;
    line-height: 0.9;
    text-align: center;
    padding-top: 0.36rem;
    transform: rotate(-4deg);
    font-weight: 600;
    font-family: var(--heading-font-family);
    font-size: 0.875rem;
    color: rgb(var(--text-color, var(--document-color)));
    position: absolute;
    left: 3rem;
    top: 2.2rem;
  }

  .compact .tag {
    left: -0.2rem;
    top: 2.2rem;
  }
</style>
