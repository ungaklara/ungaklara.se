<script>
  import { asText } from '@prismicio/client'
  import { enhance } from '$app/forms'

  import { intersection } from '$lib/utils/intersection.js'
  import SchoolBooking from '$lib/SchoolBooking.svelte'
  import Blockquote from '$lib/Blockquote.svelte'
  import Instagram from '$lib/Instagram.svelte'
  import RichText from '$lib/RichText.svelte'
  import GridCell from '$lib/GridCell.svelte'
  import Calendar from '$lib/Calendar.svelte'

  import { track } from '$lib/utils/track.js'
  import { resolve } from '$lib/prismic.js'
  import Branding from '$lib/Branding.svelte'
  import Byline from '$lib/Byline.svelte'
  import Framed from '$lib/Framed.svelte'
  import Banner from '$lib/Banner.svelte'
  import Button from '$lib/Button.svelte'
  import Signup from '$lib/Signup.svelte'
  import Embed from '$lib/Embed.svelte'
  import Intro from '$lib/Intro.svelte'
  import Html from '$lib/Html.svelte'
  import Grid from '$lib/Grid.svelte'
  import Card from '$lib/Card.svelte'
  import { asImageWidthSrcSet } from '@prismicio/helpers'

  export let data
  export let form

  $: parent = data.page.data.parent
  $: parentHref = resolve(parent)

  $: slices = data.page.data.body.reduce(function (acc, slice) {
    switch (slice.slice_type) {
      case 'link_blurb':
      case 'file_blurb':
      case 'any_blurb': {
        const prev = acc.at(-1)
        if (prev?.slice_type === '__blurbs') prev.items.push(slice)
        else acc.push({ slice_type: '__blurbs', items: [slice] })
        return acc
      }
      default:
        return acc.concat(slice)
    }
  }, [])

  const tracker = (event, items) => () => {
    track(event, { items: Array.isArray(items) ? items : [items] })
  }

  function blurbAsItem(slice) {
    switch (slice.slice_type) {
      case 'link_blurb':
        return {
          item_id: resolve(slice.primary.link),
          item_name: asText(slice.primary.link.data.title),
          item_category: 'Intern länk'
        }
      case 'file_blurb':
        return {
          item_id: slice.primary.file.url,
          item_name: asText(slice.primary.title),
          item_category: 'Länk till dokument eller bild'
        }
      case 'any_blurb':
        return {
          item_id: resolve(slice.primary.link),
          item_name: asText(slice.primary.title),
          item_category: 'Extern länk'
        }
      default:
        return null
    }
  }

  function image(props) {
    if (!props?.url) return null

    const sources = asImageWidthSrcSet(props, {widths: [200, 400, 600, 900, 1600], crop: ['focalpoint']});

    return {
      srcset: sources?.srcset,
      sizes: '(min-width: 600px) 50vw, 100vw',
      alt: props.alt || '',
      src: sources?.src,
      ...props.dimensions
    }
  }
</script>

<div class="u-container">
  <div>
    {#if !data.page.data.hide_intro}
      <header>
        <Intro title={asText(data.page.data.title)}>
          <span slot="badge">
            {#if parentHref}
              <a href={parentHref}>
                ← {parent.data.shortname?.length
                  ? asText(parent.data.shortname)
                  : asText(parent.data.title)}
              </a>
            {:else if data.page.data.shortname?.length}
              {asText(data.page.data.shortname)}
            {/if}
          </span>
          <RichText slot="text" content={data.page.data.description} />
        </Intro>
        {#if data.page.data.show_image && data.page.data.featured_image}
          {@const featured = data.page.data.featured_image}
          {@const sources = asImageWidthSrcSet(featured, {widths: [200, 400, 800], ar: '278:195'})}
          {@const image = Object.assign(
            {
              src: sources?.src,
              sizes: '15rem',
              srcset: sources?.srcset,
              alt: featured.alt || ''
            },
            featured.dimensions
          )}
          <figure class="intro-image">
            <Framed format="ellipse" {...image} />
          </figure>
        {/if}
      </header>
    {/if}

    {#each slices as slice, index}
      <div class="slice slice-{slice.slice_type}">
        {#if slice.slice_type === '__blurbs'}
          <div
            use:intersection={tracker(
              'view_item_list',
              slice.items.map(blurbAsItem)
            )}>
            <Grid size={{ md: '1of2', lg: '1of3' }}>
              {#each slice.items as item}
                {#if item.slice_type === 'link_blurb'}
                  {@const href = resolve(item.primary.link)}
                  {#if href}
                    <GridCell>
                      <Card
                        on:click={tracker('select_item', blurbAsItem(item))}
                        title={asText(item.primary.link.data?.shortname) ||
                          asText(item.primary.link.data?.title)}
                        image={image(item.primary.link.data?.featured_image)}
                        color={item.primary.color ||
                          item.primary.link.data?.theme}
                        link={{ href, text: item.primary.link.data?.cta }}>
                        <RichText
                          content={item.primary.link.data?.description} />
                      </Card>
                    </GridCell>
                  {/if}
                {/if}

                {#if item.slice_type === 'file_blurb'}
                  {#if item.primary.file.url}
                    <GridCell>
                      <Card
                        on:click={tracker('select_item', blurbAsItem(item))}
                        title={asText(item.primary.title)}
                        image={image(item.primary.image)}
                        color={item.primary.color}
                        link={{ href: item.primary.file.url }}>
                        <RichText content={item.primary.text} />
                      </Card>
                    </GridCell>
                  {/if}
                {/if}

                {#if item.slice_type === 'any_blurb'}
                  {@const href = resolve(item.primary.link)}
                  {#if href}
                    <GridCell>
                      <Card
                        on:click={tracker('select_item', blurbAsItem(item))}
                        title={asText(item.primary.title)}
                        image={item.primary ? image(item.primary.image) : null}
                        color={item.primary.color}
                        link={{ href }}>
                        <RichText content={item.primary.text} />
                      </Card>
                    </GridCell>
                  {/if}
                {/if}
              {/each}
            </Grid>
          </div>
        {/if}

        {#if slice.slice_type === 'text'}
          {#if asText(slice.primary.text) || slice.items.length}
            <div>
              {#if slice.primary.text.length}
                <Html size="large">
                  <RichText content={slice.primary.text} />
                </Html>
              {/if}
              {#if slice.items.length}
                {@const size =
                  slice.items.length > 2
                    ? { md: '1of2', lg: '1of3' }
                    : { md: '1of2' }}
                <Grid {size} class="u-spaceMd">
                  {#each slice.items as item}
                    <GridCell>
                      <Html size="large">
                        <RichText content={item.text} />
                      </Html>
                    </GridCell>
                  {/each}
                </Grid>
              {/if}
            </div>
          {/if}
        {/if}

        {#if slice.slice_type === 'heading'}
          {@const heading = asText(slice.primary.heading)}
          {#if heading}
            {#if slice.primary.divider}
              <div class="u-divider u-spaceLg" />
            {/if}
            <Html
              size="large"
              class={slice.primary.divider ? 'u-spaceSm' : 'u-spaceLg'}>
              <h2>{heading}</h2>
              <RichText content={slice.primary.text} />
            </Html>
          {/if}
        {/if}

        {#if slice.slice_type === 'quote'}
          <div class="u-spaceLg">
            <Blockquote>
              <div slot="text"><RichText content={slice.primary.text} /></div>
              <div slot="cite"><RichText content={slice.primary.cite} /></div>
            </Blockquote>
          </div>
        {/if}

        {#if slice.slice_type === 'branding'}
          <Branding />
        {/if}

        {#if slice.slice_type === 'image'}
          {#if slice.primary.image.url}
            {@const sources = asImageWidthSrcSet(slice.primary.image, {widths: [
              600,
              900,
              1600,
              3000
            ]})}
            <figure>
              <Html
                size="large"
                class={slice.primary.smaller ? '' : 'u-sizeFull'}>
                <img
                  sizes="100vw"
                  srcset={sources?.srcset}
                  src={sources?.src}
                  alt={slice.primary.image.alt || ''}
                  {...slice.primary.image.dimensions} />
                {#if slice.primary.image.copyright}
                  <figcaption>
                    <small class="muted">{slice.primary.image.copyright}</small>
                  </figcaption>
                {/if}
              </Html>
            </figure>
          {/if}
        {/if}

        {#if slice.slice_type === 'pjas_hypebild_'}
          {#if slice.primary.image.url}
            {@const title = asText(slice.primary.name)}
            {@const link = resolve(slice.primary.link)}
            {@const tag = slice.primary.tag}
            {@const desc = slice.primary.desc}
            {@const sources = asImageWidthSrcSet(slice.primary.image, {
              widths: [
                600,
                900,
                1600,
                3000,
              ]
            })}
            <Banner
              top={index === 0}
              {title}
              {link}
              {tag}
              {desc}
              image={slice.primary.image.url
                ? {
                    src: sources?.src,
                    sizes: '90vw',
                    srcset: sources?.srcset,
                    ...slice.primary.image.dimensions
                  }
                : null} />
          {/if}
        {/if}

        {#if slice.slice_type === 'video'}
          {@const items = slice.items.filter((item) => item.video.embed_url)}
          {#if slice.primary.video.embed_url}
            <Embed content={slice.primary.video} />
          {/if}
          {#if items.length}
            <div class="u-spaceMd u-md-uncontain">
              <Grid
                carousel
                size={{ md: `1of${items.length - 1 < 3 ? 2 : 3}` }}>
                {#each items as item}
                  <GridCell>
                    <Embed content={item.video} />
                  </GridCell>
                {/each}
              </Grid>
            </div>
          {/if}
        {/if}

        {#if slice.slice_type === 'author' || slice.slice_type === 'contact'}
          {@const sources = asImageWidthSrcSet(slice.primary.image, {widths: [200, 400, 800], ar: '278:195'})}
          <Byline
            heading={asText(slice.primary.heading)}
            image={slice.primary.image.url
              ? Object.assign(
                  {
                    src: sources?.src,
                    sizes: '15rem',
                    srcset: sources?.srcset,
                    alt: slice.primary.image.alt || ''
                  },
                  slice.primary.image.dimensions
                )
              : null}>
            <RichText content={slice.primary.text} />
          </Byline>
        {/if}

        {#if slice.slice_type === 'accordion'}
          {#each slice.items as item}
            {#if item.heading.length && item.text}
              <Html size="large" class="u-sizeFull">
                <details>
                  <summary><h3>{asText(item.heading)}</h3></summary>
                  <RichText content={item.text} />
                </details>
              </Html>
            {/if}
          {/each}
        {/if}

        {#if slice.slice_type === 'team'}
          {#if slice.items.length}
            <Grid
              size={{
                lg: '1of4',
                md: '1of2',
                xs: slice.items.find((item) => item.image.url) ? '1of2' : '1of1'
              }}>
              {#each slice.items as item}
                <GridCell>
                  <article>
                    <Html>
                      {#if item.image.url}
                        {@const sources = asImageWidthSrcSet(
                          item.image,
                          {widths: [200, 400, 800], ar: '7:5'},
                        )}
                        <img
                          class="u-sizeFull"
                          sizes="13em"
                          srcset={sources?.srcset}
                          style="max-width: 13em !important; width: 100%"
                          alt={item.image.alt || ''}
                          src={sources?.src}
                          {...item.image.dimensions} />
                      {/if}
                      <div class="u-nudgeMd">
                        {#if item.label}
                          <strong class="u-textLabel">{item.label}</strong>
                        {/if}
                        <RichText content={item.text} />
                      </div>
                    </Html>
                  </article>
                </GridCell>
              {/each}
            </Grid>
          {/if}
        {/if}

        {#if slice.slice_type === 'button'}
          {#if slice.primary.text && slice.primary.link}
            <Button primary href={resolve(slice.primary.link)}>
              {slice.primary.text}
            </Button>
          {/if}
        {/if}

        {#if slice.slice_type === 'signup'}
          <Signup result={form?.signup}>
            <div slot="primary">
              <Html size="large">
                <h2 class="h1">{asText(data.settings.data.signup_heading)}</h2>
                <RichText content={data.settings.data.signup_content} />
              </Html>
            </div>
            <div slot="success">
              <Html>
                <RichText content={data.settings.data.signup_success} />
              </Html>
            </div>
          </Signup>
        {/if}

        {#if slice.slice_type === 'resources'}
          {@const heading = asText(slice.primary.heading)}
          <hr
            style="margin: 2.5rem 0;"
            use:intersection={tracker(
              'view_item_list',
              slice.items.map((item) => ({
                item_list_name: heading,
                item_id: item.file.url,
                item_name: item.name,
                item_category: 'Pedagogiskt material'
              }))
            )} />
          {#if heading || asText(slice.primary.description)}
            <Html>
              <h2>{heading}</h2>
              <RichText content={slice.primary.description} />
            </Html>
          {/if}
          <Grid
            class="u-spaceMd"
            size={{ sm: '1of2', md: '1of2', lg: '1of3', xl: '1of4' }}>
            {#each slice.items as item}
              <GridCell>
                <Card
                  on:click={tracker('select_item', {
                    item_list_name: heading,
                    item_id: item.file.url,
                    item_name: item.name,
                    item_category: 'Pedagogiskt material'
                  })}
                  size="small"
                  title={item.name}
                  image={image(item.image) ||
                    image(slice.primary.event?.data?.poster)}
                  square
                  color={slice.primary.event?.data?.theme}
                  link={{ href: item.file.url, text: 'Ladda ner' }} />
              </GridCell>
            {/each}
          </Grid>
        {/if}

        {#if slice.slice_type === 'school_booking_form'}
          {@const hasBody = Boolean(asText(slice.primary.text))}
          <form class="u-spaceLg" action="?/booking" method="POST" use:enhance>
            <SchoolBooking result={form?.booking}>
              {#if hasBody}
                <Html>
                  <RichText content={slice.primary.text} />
                </Html>
              {/if}
            </SchoolBooking>
          </form>
        {/if}

        {#if slice.slice_type === 'upcoming_shows'}
          {@const events = data[slice.id]}
          {@const text = asText(slice.primary.text)}
          {#if events}
            <div class="u-spaceLg">
              <div class="header">
                <Html>
                  <RichText content={slice.primary.text} />
                  <a
                    style="position: absolute; top: 0.5rem; right: 0;"
                    href="/scen/kalendarium">
                    Hela kalendariet
                  </a>
                </Html>
              </div>
              <div class:u-spaceMd={text}>
                <Calendar compact {events} limit={6} />
              </div>
            </div>
          {/if}
        {/if}

        {#if slice.slice_type === 'instagram-bilder'}
          {@const posts = slice.items}
          {@const text = asText(slice.primary.heading)}
          {#if posts}
            <div class="u-spaceLg">
              <Html>
                <RichText content={slice.primary.heading} />
              </Html>
              <div class:u-spaceMd={text}>
                <Instagram {posts} />
              </div>
            </div>
          {/if}
        {/if}
      </div>
    {/each}
  </div>
</div>

<style>
  .slice {
    position: relative;
    margin-top: var(--space-md);
  }

  .slice-signup,
  .slice-branding,
  .slice-pjas_hypebild_:not(:first-child) {
    margin-top: var(--space-xl);
  }

  .slice-branding + * {
    margin-top: var(--space-xl);
  }

  .slice-heading + .slice-button {
    margin-top: var(--space-sm);
  }

  .slice-text + .slice-author,
  .slice-button + .slice-author {
    margin-top: var(--space-lg);
  }

  header + .slice-button {
    margin-top: var(--space-sm);
  }

  @media (min-width: 700px) {
    .slice-pjas_hypebild_:first-child {
      margin-top: 0;
    }

    .slice-pjas_hypebild_ {
      margin-left: calc(var(--document-margin) * -1);
      margin-right: calc(var(--document-margin) * -1);
    }
  }

  .intro-image {
    display: none;
  }

  @media (width > 600px) {
    .intro-image {
      display: block;
      position: absolute;
      width: clamp(9rem, 26vw, 20rem);
      top: clamp(8rem, 100vh, 10rem);
      right: var(--document-margin);
    }
  }

  @media (width > 1300px) {
    .intro-image {
      right: 8rem;
    }
  }

  .link {
    display: inline-block;
    font-size: 1rem;
    font-family: var(--heading-font-family);
    font-weight: 600;
    white-space: nowrap;
    user-select: none;
    margin: 1rem 0 -1rem;
  }
</style>
