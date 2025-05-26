<script context="module" lang="ts">
    import LiteYouTubeEmbed from '$lib/LiteYouTubeEmbed.svelte';
    import { asImageWidthSrcSet } from '@prismicio/helpers';

    // match short and long youtube links
    // https://www.youtube.com/watch?foo=bar&v=WwE7TxtoyqM&bin=baz
    // https://youtu.be/gd6_ZECm58g
    export const YOUTUBE_RE =
        /https?:\/\/(?:www.)?youtu\.?be(?:\.com\/watch\?(?:.*?)v=|\/)(.+?)(?:&|$)/;

    export function getId(props) {
        switch (props?.provider_name) {
            case 'YouTube': {
                const match = props.embed_url.match(YOUTUBE_RE);
                return match ? match[1] : null;
            }
            case 'Vimeo': {
                const match = props.embed_url.match(
                    /vimeo\.com\/(.+?)(?:\?|$)/,
                );
                return match ? match[1] : null;
            }
            default:
                return null;
        }
    }

    export function getThumbnailUrl(props) {
        switch (props?.provider_name) {
            case 'YouTube': {
                return (
                    props?.thumbnail_url?.replace(
                        /\/?\w+.jpg$/,
                        `/maxresdefault.jpg`,
                    ) ?? props?.thumbnail_url
                );
            }
            // case 'Vimeo': {
            //   const match = props.embed_url.match(/vimeo\.com\/(.+?)(?:\?|$)/)
            //   return match ? match[1] : null
            // }
            default:
                return props?.thumbnail_url;
        }
    }

    export function getUrl(props) {
        let url = props.embed_url;
        if (/youtu\.be/.test(url)) {
            url = url.replace(/youtu\.be\/(\w+)$/, 'youtube.com/watch?v=$1');
        }
        if (/youtube/.test(url)) {
            const id = url.match(/youtube\.com\/watch\?v=(.+)?\??/)[1];
            return `https://www.youtube.com/embed/${id}?autoplay=1&rel=0&showinfo=0`;
        } else if (/vimeo/.test(url)) {
            const id = url.match(/vimeo\.com\/(.+)?\??/)[1];
            return `https://player.vimeo.com/video/${id}?badge=0&autoplay=1`;
        }
        return url;
    }
</script>

<script>
    /** @type {object?} */
    export let content = null;

    $: id = getId(content);
    $: provider = content?.provider_name.toLowerCase();
    $: thumbnailUrl = getThumbnailUrl(content);
    $: thumbnailProps = {
        url: thumbnailUrl,
        dimensions: {
            width: content?.thumbnail_width,
            height: content?.thumbnail_height,
        },
        alt: `${content?.title} thumbnail`,
        copyright: null,
    };
    $: thumbnail = asImageWidthSrcSet(thumbnailProps, {
        widths: [400, 900, 1800, 2600],
        // trim: 'auto',
    });
</script>

{#if id}
    {#if provider === 'youtube'}
        <figure class="embed">
            <LiteYouTubeEmbed
                videoId={id}
                posterSrc={thumbnail.src}
                posterSrcSet={thumbnail.srcset}
                posterWidth={content?.thumbnail_width}
                posterHeight={content?.thumbnail_height}>
                <button
                    type="button"
                    class="lite-playbtn"
                    aria-label={`Spela ${content.title || ''}`} />
            </LiteYouTubeEmbed>
        </figure>
    {:else if provider === 'vimeo'}
        <script
            type="module"
            src="https://cdn.jsdelivr.net/npm/lite-vimeo-embed/+esm"></script>
        <lite-vimeo
            videoid={id}
            class="embed"
            >
            <button
                type="button"
                class="lite-playbtn"
                aria-label={`Spela ${content.title || ''}`} />
        </lite-vimeo>
    {/if}
{/if}

<style>
    .embed {
        width: 100%;
        position: relative;
        color: #fff;
        user-select: none;
        border-radius: var(--border-radius);
        overflow: hidden;
    }

    .lite-playbtn {
        display: block;
        width: 0;
        height: 0;
        cursor: default;
    }

    :global(.ltv-playbtn) {
      display: none;
    }

    :global(.ltv-activated) :global(.lite-playbtn) {
        display: none;
    }

    .lite-playbtn::before {
        content: '';
        position: absolute;
        inset: 0;
    }

    .lite-playbtn::after {
        content: '';
        width: 3rem;
        height: 3rem;

        position: absolute;
        left: 50%;
        top: 50%;
        z-index: 1;

        text-indent: 3rem;
        white-space: nowrap;
        overflow: hidden;

        transform: translate(-50%, -50%);
        transition: transform 140ms var(--ease);
        will-change: transform;
        background-color: rgba(var(--current-color));
        background-image: url('/icon.svg');
        background-size: 100% 100%;
        border-radius: 100%;
        box-shadow: 0 0 2rem 1px rgba(0, 0, 0, 0.15);
    }

    .Embed:hover .lite-playbtn::after {
        transform: translate(-50%, -50%) scale(1.08);
    }

    .fullscreen::after,
    .hidden::after {
        content: none;
    }

    @media (min-width: 600px) {
        .Embed:not(.small) .lite-playbtn::after {
            width: 4.5rem;
            height: 4.5rem;
            transform: translate(-50%, -60%);
        }

        .Embed:not(.small):hover .lite-playbtn::after {
            transform: translate(-50%, -60%) scale(1.08);
        }
    }

    .image {
        display: block;
        width: 100%;
        height: auto;
    }

    @supports (object-fit: cover) {
        .Embed:not(.small) .image {
            min-height: 40vh;
            object-fit: cover;
        }
    }

    .caption {
        display: none;
        flex-direction: column-reverse;
        width: 100%;
        min-height: 25%;
        padding: 1.4rem 1.4rem;
        position: absolute;
        bottom: 0;
        left: 0;
        background-image: linear-gradient(
            rgba(0, 0, 0, 0),
            rgba(0, 0, 0, 0.3) 40%,
            rgba(0, 0, 0, 0.5)
        );
    }

    @media (min-width: 600px) {
        .Embed:not(.small) .caption {
            display: flex;
        }
    }

    .title {
        padding-top: 0.2rem;
        position: relative;
        z-index: 1;
        line-height: 1.15;
        font-size: 1.2rem;
        font-family: var(--heading-font-family);
        letter-spacing: var(--heading-letter-spacing);
        word-spacing: var(--heading-word-spacing);
    }

    @media (min-width: 1000px) {
        .title {
            font-size: 1.75rem;
            line-height: 1.1;
        }

        .caption {
            padding: 1.4rem 2rem;
        }
    }

    /**
 * Fullscreen mode
 */

    .fullscreen {
        width: 100vw;
        height: 100vh;
        padding: 2rem;

        position: fixed;
        top: 0;
        left: 0;
        z-index: 30;

        background: rgba(0, 0, 0, 0.85);
        animation: -appear 500ms cubic-bezier(0.165, 0.84, 0.44, 1) forwards;
        border-radius: 0;
    }

    @keyframes -appear {
        from {
            opacity: 0;
        }

        to {
            opacity: 1;
        }
    }

    .close {
        position: absolute;
        inset: 0;
        z-index: -2;
    }

    .cross {
        box-sizing: content-box;
        width: 1.5rem;
        height: 1.5rem;
        padding: 1rem;
        position: absolute;
        top: 2rem;
        right: 2rem;
        border-radius: 2px;
        pointer-events: none;
        background-color: rgba(81, 81, 81, 0);
        transition: background-color 180ms var(--ease-out);
    }

    .cross::before,
    .cross::after {
        content: '';
        width: 1.5rem;
        height: 2px;
        position: absolute;
        right: 1rem;
        top: 50%;
        background: currentcolor;
    }

    .cross::before {
        transform: rotate(45deg);
    }

    .cross::after {
        transform: rotate(-45deg);
    }

    .close:hover .cross {
        background-color: rgba(81, 81, 81, 1);
    }

    .wrapper {
        height: 0;
        width: 88%;
        max-width: var(--document-max-width);

        position: absolute;
        left: 50%;
        top: 50%;
        z-index: -1;
    }

    @media (min-width: 600px) {
        .wrapper {
            width: 75%;
        }
    }

    .iframe {
        height: 0;
        width: 100%;
        position: absolute;
        left: 0;
        top: 0;
        padding-bottom: calc(100% * (9 / 16));

        background-color: #000;
        box-shadow: 0 0 10rem rgba(0, 0, 0, 0.2);
        transform: translate(-50%, -50%);
        will-change: transform;
    }

    .iframe iframe {
        width: 100%;
        height: 100%;
        position: absolute;
        inset: 0;
    }

    .fullscreen .iframe iframe {
        visibility: hidden;
        animation: content--appear 0ms 650ms linear forwards;
    }

    @keyframes content--appear {
        to {
            visibility: visible;
        }
    }
</style>
