<script>
    import '@beyonk/gdpr-cookie-consent-banner/banner.css';
    import GdprBanner from '@beyonk/gdpr-cookie-consent-banner';
    import Analytics from '$lib/Analytics.svelte';

    let consents = {necessary: false};

    $: hasConsent = () => !!consents?.analytics;
    $: showCookieBanner = () => !consents?.necessary;
    
    $: if (typeof localStorage !== 'undefined') {
        consents = JSON.parse(localStorage.getItem('consents') || JSON.stringify(consents));
    }

    let gdprProps = {
        cookieName: 'ungaklara',
        cookieConfig: {
            domain: 'ungaklara.se',
            path: '/',
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
                value: true,
            },
            // tracking: false,
            tracking: {
                label: 'Cookies för profilering',
                description: 'Används för att följa .',
                value: true,
            },
            analytics: {
                label: 'Cookies för trafikanalys',
                description:
                    'Används för att hantera Google Analytics, en tjänst från Google.',
                value: consents?.analytics ?? true,
            },
            // marketing: false
        },
        showEditIcon: false,
    };

    function handleConsentUpdate(event) {
        if (typeof localStorage === 'undefined') return;

        consents.necessary = true;

        consents = {
            ...consents,
            ...{
                [event.type]: event.detail.agreed,
            },
        };
        
        localStorage.setItem('consents', JSON.stringify(consents));
    }
</script>

<svelte:head>
    <Analytics doTrack={() => hasConsent()} />
</svelte:head>

{#if showCookieBanner()}
    <GdprBanner {...gdprProps} on:analytics={handleConsentUpdate} />
{/if}
