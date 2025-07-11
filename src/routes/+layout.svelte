<script>
    import { asText } from '@prismicio/client';
    import { onMount } from 'svelte';

    import { resolve } from '$lib/prismic.js';
    import Header from '$lib/Header.svelte';
    import Footer from '$lib/Footer.svelte';
    import Meta from '$lib/Meta.svelte';
    import CookieBanner from '$lib/CookieBanner.svelte';
    import { YBUG_CONFIG } from '$lib/utils/constants.js';

    export let data;

    let vma = true;

    $: settings = data.settings.data;

    onMount(() => {
        navigator.serviceWorker.getRegistrations().then((workers) => {
            for (const worker of workers) {
                // Unregister any lingering service workers from old app versions
                if (worker.active?.scriptURL?.endsWith('sw.js')) {
                    worker.unregister();
                }
            }
        });
    });
</script>

<svelte:head>
    <Meta />

    <script>
        document.documentElement.setAttribute('scripting-enabled', '');
        window.onerror = function () {
            document.documentElement.removeAttribute('scripting-enabled');
            document.documentElement.setAttribute('scripting-initial-only', '');
        };
    </script>

    {#if data.previewToken}
        <script
            async
            defer
            src="https://static.cdn.prismic.io/prismic.js?new=true&repo=unga-klara"></script>
    {/if}
</svelte:head>

<CookieBanner />

<div class="layout">
    <div class="gradient" />
    <Header items={settings.header_menu.map((item) => item.primary)}>
        {#if asText(settings.vma_heading) && vma}
            <div class="u-container">
                <div class="vma">
                    <h3>
                        {asText(settings.vma_heading)}

                        {#if settings.vma_link && resolve(settings.vma_link)}
                            <a href={resolve(settings.vma_link)}>Läs mer</a>
                        {/if}
                    </h3>
                    <button class="close" on:click={() => (vma = false)}>
                        Stäng
                    </button>
                </div>
            </div>
        {/if}
    </Header>
    <div class="main">
        <slot />
    </div>
    <Footer {settings} />
    <svelte:element this="script">
        {@html `
        (function() {
            window.ybug_settings = ${JSON.stringify(YBUG_CONFIG)};
            var ybug = document.createElement('script'); ybug.type = 'text/javascript'; ybug.async = true;
            ybug.src = 'https://widget.ybug.io/button/'+window.ybug_settings.id+'.js';
            var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ybug, s);
        })();
        `}
    </svelte:element>
</div>

<style>
    @import '$lib/index.css';

    .layout {
        display: flex;
        flex-direction: column;
        min-height: 100%;
        position: relative;
        min-width: var(--document-min-width);
        overflow-x: hidden;
    }

    .main {
        flex-grow: 1;
        max-width: 100%;
        min-height: 100vh;
    }

    .gradient {
        width: 100%;
        height: calc(100% - 200vh);
        position: absolute;
        top: 100vh;
        left: 0;
        z-index: -1;
        background: rgb(var(--document-background));
    }

    .gradient::before,
    .gradient::after {
        content: '';
        width: 100%;
        height: 100vh;
        position: absolute;
        top: calc(100vh * -1);
        z-index: 0;
        background: linear-gradient(
            rgba(var(--document-background), 0) 0%,
            rgba(var(--document-background), 1) 100%
        );
        pointer-events: none;
    }

    .gradient::after {
        height: 100vh;
        top: calc(100% - 1px);
        transform: rotate(180deg);
    }

    .vma {
        padding: 1.25rem 4.5rem 1.25rem 1.25rem;
        background: black;
        border-radius: var(--border-radius);
        color: #fff;
        margin-top: 1rem;
        font-family: var(--heading-font-family);
        letter-spacing: var(--heading-letter-spacing);
        word-spacing: var(--heading-word-spacing);
        line-height: 1.3;
        position: relative;
    }

    .vma h3 {
        display: flex;
        flex-wrap: wrap;
        gap: 0.5em 1em;
        max-width: 55em;
    }

    .vma a {
        text-underline-offset: 0.25em;
        text-decoration-thickness: var(--border-width);
        text-decoration: underline;
        white-space: nowrap;
    }

    .vma .close {
        font-size: 0;
        color: transparent;
        line-height: 0.5rem;
        top: 0.25rem;
        position: absolute;
        cursor: pointer;
        padding: 0.5rem;
        top: 0.9rem;
        right: 1rem;
        width: 2rem;
        height: 2rem;
        background: #fff;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .vma .close::after {
        display: block;
        content: '⨉';
        font-size: 1.25rem;
        color: #000;
        position: relative;
        top: -0.2rem;
    }

    @media (min-width: 800px) {
        .vma {
            font-size: 1.25rem;
            padding: 1.5rem 1.75rem;
            margin-top: 2rem;
        }

        .vma .close {
            top: 1.3rem;
            right: 1.5rem;
        }
    }

    :global(
            .cookie-consent-banner::part(consent--description),
            .cookieConsent__Description
        ) {
        margin-top: 0;
    }

    :global(.cookieConsent__Button) {
        display: inline-flex;
        align-items: center;
        padding: 0 1rem 0.2rem;
        border: var(--border-width) solid rgb(var(--document-color));
        position: relative;
        font-size: 1rem;
        color: #fff;
        background: transparent;
        font-weight: 600;
        white-space: nowrap;
        user-select: none;
        text-align: center;
        font-family: var(--heading-font-family);
        letter-spacing: var(--heading-letter-spacing);
        word-spacing: var(--heading-word-spacing);
        border-radius: var(--border-radius);
        height: 3rem;
        cursor: pointer;
    }

    :global(.cookieConsent__Button[type='submit']) {
        color: #000;
        background: #fff;
    }

    :global(.cookieConsent__Button[type='button']) {
        padding-left: 0;
        border-left: 0;
    }

    :global(
            .cookie-consent-banner::part(operations--list),
            .cookieConsentOperations__List
        ) {
        border-radius: var(--border-radius);
        margin: 1rem;
    }

    :global(.cookieConsentOperations__Item label) {
        display: block;
        padding-bottom: 0.35rem;
        text-decoration: none;
        font-size: 1.25rem !important;
        font-family: var(--heading-font-family);
        line-height: 1.07;
        margin-top: 0.25rem;
        letter-spacing: var(--heading-letter-spacing);
        word-spacing: var(--heading-word-spacing);
        text-wrap: balance;
    }

    :global(.cookieConsent__Button--Close) {
        margin-top: 0.5rem;
    }

    :global(.cookieConsentOperations) {
        align-items: center;
        justify-content: center;
        transition: none;
        user-select: none;
    }

    :global(.cookieConsentWrapper) {
        user-select: none;
        padding: 0;
    }

    :global(.cookieConsent) {
        padding: 20px;
    }
</style>
