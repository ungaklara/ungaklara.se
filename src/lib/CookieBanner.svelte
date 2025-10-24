<script>
  import '@beyonk/gdpr-cookie-consent-banner/banner.css'
  import GdprBanner from '@beyonk/gdpr-cookie-consent-banner'
  import { dev } from '$app/environment'
  import { GOOGLE_TAG_MANAGER_ID } from './utils/constants'

  let consents = { necessary: true }

  let gdprProps = {
    cookieName: 'ungaklara',
    cookieConfig: {
      domain: dev ? 'localhost' : 'ungaklara.se',
      path: '/'
    },
    heading: 'Snabb fråga om cookies',
    description:
      'De används för att förbättra hemsidan och för att samla besöksstatistik.',
    acceptLabel: 'Godkänn',
    acceptAllLabel: 'Godkänn alla',
    acceptSelectedLabel: 'Godkänn valda',
    rejectLabel: 'Endast nödvändiga',
    settingsLabel: 'Välj vilka',
    closeLabel: 'Stäng',
    editLabel: 'Ändra inställningar',
    choices: {
      necessary: {
        label: 'Nödvändiga kakor',
        description:
          'Används för att vi inte ska fråga dig om cookie-inställningar igen.',
        value: true
      },
      analytics: {
        label: 'Cookies för trafikanalys',
        description:
          'Används för att hantera Google Analytics, en tjänst från Google.',
        value: true
      },
      // ad_storage, ad_personalization, ad_user_data
      marketing: {
        label: 'Cookies för marknadsföring',
        description: 'Används för marknadsföringsdata.',
        value: true
      },
      tracking: false
      // {
      //   label: 'Cookies för profilering',
      //   description: 'Används för reklam.',
      //   value: true
      // }
    },
    showEditIcon: false
  }

  const activeConsentOptions = JSON.stringify(
    Object.entries(gdprProps.choices)
      .filter(([key, value]) => !!value)
      .map(([key, value]) => key)
  )

  function pushConsentToGoogle(consent) {
    const fullfilledConsents = JSON.stringify(Object.keys(consent))
    const allConsentsFullfilled = fullfilledConsents === activeConsentOptions

    if (typeof window.gtag !== 'undefined' && allConsentsFullfilled) {
      const googleConsents = {
        ad_user_data: !!consent.marketing ? 'granted' : 'denied',
        ad_personalization: !!consent.marketing ? 'granted' : 'denied',
        ad_storage: !!consent.marketing ? 'granted' : 'denied',
        analytics_storage: !!consent.analytics ? 'granted' : 'denied'
      }

      window.gtag('consent', 'update', googleConsents)
    }
  }

  function handleConsentUpdate(event) {
    if (typeof localStorage === 'undefined') return

    consents = {
      ...consents,
      ...{
        [event.type]: event.detail.agreed
      }
    }

    pushConsentToGoogle(consents)
  }
</script>

<svelte:head>
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
      gtag('consent', 'default', {
        'ad_user_data': 'denied',
        'ad_personalization': 'denied',
        'ad_storage': 'denied',
        'analytics_storage': 'denied',
        'wait_for_update': 500,
      });
      gtag('js', new Date())
      gtag('config', '${GOOGLE_TAG_MANAGER_ID}')
    `}
  </svelte:element>
</svelte:head>

<GdprBanner
  {...gdprProps}
  on:analytics={handleConsentUpdate}
  on:marketing={handleConsentUpdate} />
