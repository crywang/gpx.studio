<script lang="ts">
    import '../app.pcss';
    import { ModeWatcher } from 'mode-watcher';
    import { isLoading, _, locale } from 'svelte-i18n';
    import { page } from '$app/stores';
    import Nav from '$lib/components/Nav.svelte';
    import Footer from '$lib/components/Footer.svelte';
    import { onMount } from 'svelte';
    import { convertOldEmbeddingOptions } from '$lib/components/embedding/Embedding';
    import { base } from '$app/paths';
    import { languages } from '$lib/languages';
    import { browser } from '$app/environment';
    import { goto } from '$app/navigation';
    import { getURLForLanguage } from '$lib/utils';

    export let data: {
        guideTitles: Record<string, string>;
    };

    const appRoutes = ['/[[language]]/app', '/[[language]]/embed'];

    onMount(() => {
        if ($page.url.searchParams.has('embed')) {
            // convert old embedding options to new format and redirect to new embed page
            let folders = $page.url.pathname.split('/');
            let locale =
                folders.indexOf('l') >= 0 ? (folders[folders.indexOf('l') + 1] ?? 'en') : 'en';
            window.location.href = `${getURLForLanguage(locale, '/embed')}?options=${encodeURIComponent(JSON.stringify(convertOldEmbeddingOptions($page.url.searchParams)))}`;
        }
    });

    $: if ($page.route.id?.includes('[[language]]')) {
        if ($page.params.language) {
            let lang = $page.params.language.replace('/', '');
            if ($locale !== lang) {
                if (languages.hasOwnProperty(lang)) {
                    $locale = lang;
                } else if (browser) {
                    goto(`${base}/404`);
                }
            }
        } else if ($locale !== 'en') {
            $locale = 'en';
        }
    }

    $: if (browser && !$isLoading && $locale) {
        let title = `gpx.studio — ${$_(`metadata.${$page.route.id?.replace('/[[language]]', '').split('/')[1] ?? 'home'}_title`)}`;
        if ($page.params.guide) {
            document.title = `${title} | ${data.guideTitles[$page.params.guide]}`;
        } else {
            document.title = title;
        }
    }
    $: showNavAndFooter = $page.route.id === null || !appRoutes.includes($page.route.id);
</script>

<ModeWatcher />

<div class="flex flex-col min-h-screen">
    {#if !$isLoading}
        {#if showNavAndFooter}
            <Nav />
        {/if}
        <main class="grow flex flex-col">
            <slot />
        </main>
        {#if showNavAndFooter}
            <Footer />
        {/if}
    {/if}
</div>
