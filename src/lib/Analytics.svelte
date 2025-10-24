<script>
  import { dev } from '$app/environment'
  import { GOOGLE_TAG_MANAGER_ID } from './utils/constants'

  // export let doTrack
</script>

<svelte:element
  this="script"
  async
  src={`https://www.googletagmanager.com/gtag/js?id=${GOOGLE_TAG_MANAGER_ID}`} />

<svelte:element this="script">
  {@html `
    // Google Analytics Code
    window.dataLayer = window.dataLayer || []
    function gtag() {
        dataLayer.push(arguments)
    }
    gtag('js', new Date())
    gtag('config', '${GOOGLE_TAG_MANAGER_ID}${dev ? ', {debug_mode: true}' : ''}')
  `}
</svelte:element>

<!-- {#if doTrack()}

    <svelte:element this="script">
        {@html `
        //  Meta Pixel Code
        !(function (f, b, e, v, n, t, s) {
            if (f.fbq) return
            n = f.fbq = function () {
            n.callMethod
                ? n.callMethod.apply(n, arguments)
                : n.queue.push(arguments)
            }
            if (!f._fbq) f._fbq = n
            n.push = n
            n.loaded = !0
            n.version = '2.0'
            n.queue = []
            t = b.createElement(e)
            t.async = !0
            t.src = v
            s = b.getElementsByTagName(e)[0]
            s.parentNode.insertBefore(t, s)
        })(
            window,
            document,
            'script',
            'https://connect.facebook.net/en_US/fbevents.js'
        )
        fbq('init', '1125905244708947')
        fbq('track', 'PageView')

        // TikTok Pixel Code
        !function (w, d, t) {
            w.TiktokAnalyticsObject=t;var ttq=w[t]=w[t]||[];ttq.methods=["page","track","identify","instances","debug","on","off","once","ready","alias","group","enableCookie","disableCookie","holdConsent","revokeConsent","grantConsent"],ttq.setAndDefer=function(t,e){t[e]=function(){t.push([e].concat(Array.prototype.slice.call(arguments,0)))}};for(var i=0;i<ttq.methods.length;i++)ttq.setAndDefer(ttq,ttq.methods[i]);ttq.instance=function(t){for(
        var e=ttq._i[t]||[],n=0;n<ttq.methods.length;n++)ttq.setAndDefer(e,ttq.methods[n]);return e},ttq.load=function(e,n){var r="https://analytics.tiktok.com/i18n/pixel/events.js",o=n&&n.partner;ttq._i=ttq._i||{},ttq._i[e]=[],ttq._i[e]._u=r,ttq._t=ttq._t||{},ttq._t[e]=+new Date,ttq._o=ttq._o||{},ttq._o[e]=n||{};n=document.createElement("script")
        ;n.type="text/javascript",n.async=!0,n.src=r+"?sdkid="+e+"&lib="+t;e=document.getElementsByTagName("script")[0];e.parentNode.insertBefore(n,e)};
            ttq.load('CSVJSVJC77UA9SBJKHT0');
            ttq.page();
        }(window, document, 'ttq');
        `}
    </svelte:element>
{/if} -->
