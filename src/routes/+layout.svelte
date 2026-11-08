<script lang="ts">
	import 'beercss/dist/cdn/beer.min.css';
	import 'beercss/dist/cdn/beer.min.js';
	import '$lib/global.scss';

	import ogImage from '$lib/assets/open-graph.png';

	import { register, init, getLocaleFromNavigator, _, locale, locales } from 'svelte-i18n';
	import Icon from '$lib/components/Icon.svelte';
	import { mdiClose, mdiMenuDown, mdiWeatherNight, mdiWeatherSunny } from '@mdi/js';
	import { onMount } from 'svelte';

	const languageNames: Record<string, string> = {
		en: 'Русский',
		'zh-Hans': '简体中文 (帕夏)'
	};

	async function setup() {
		register('en', () => import('$lib/locales/en.json'));
		register('zh-Hans', () => import('$lib/locales/zh-Hans.json'));

		function getFinalInitialLocale() {
			let zhForm,
				orgNavLang = getLocaleFromNavigator() || '',
				navLang = orgNavLang.toLowerCase();
			if (navLang !== null && navLang.startsWith('zh')) {
				if (
					navLang.indexOf('-tw') !== -1 ||
					navLang.indexOf('-hk') !== -1 ||
					navLang.indexOf('-mo') !== -1 ||
					navLang.indexOf('-hant') !== -1
				) {
					zhForm = 'zh-Hant';
				} else {
					zhForm = 'zh-Hans';
				}
			}
			return zhForm || 'en';
		}

		return await Promise.allSettled([
			init({
				fallbackLocale: 'zh-Hans',
				initialLocale: getFinalInitialLocale()
			})
		]);
	}

	const setupResult = setup();

	let { children } = $props();

	let darkmode = $state(true);

	let announcementActive = $state(false);

	onMount(() => {
		if (localStorage.getItem('lastAnnouncement') != 'designFeedback') {
			announcementActive = true;
		}

		if (localStorage.getItem('darkmode') === 'false') {
			darkmode = false;
		}

		$effect(() => {
			if (darkmode) {
				document.body.classList.add('dark');
				localStorage.setItem('darkmode', 'true');
			} else {
				document.body.classList.remove('dark');
				localStorage.setItem('darkmode', 'false');
			}
		});
	});

	function hideAnnouncement() {
		localStorage.setItem('lastAnnouncement', 'designFeedback');
		announcementActive = false;
	}
</script>

<svelte:head>
	<title>Pasha Anomaly</title>
	<meta property="og:type" content="website" />
	<meta property="og:title" content="Pasha Anomaly" />
	<meta property="og:image" content={ogImage} />
	<meta
		property="og:description"
		content="Отслеживай истинный процент исследования мира Genshin Impact от Паши (свыше 100%)"
	/>
	<meta name="theme-color" content="#7c3aed" />
</svelte:head>

{#await setupResult then}
	<header>
		<nav>
			<h6 class="max"><a href="/Pasha-Anomaly/">Pasha Anomaly</a></h6>

			<label class="checkbox icon">
				<input type="checkbox" bind:checked={darkmode} />
				<span>
					<Icon d={mdiWeatherSunny}></Icon>
					<Icon d={mdiWeatherNight}></Icon>
				</span>
			</label>

			<div class="field suffix border small lang-select">
				<select bind:value={$locale}>
					{#each $locales as l}
						<option value={l}>{languageNames[l]}</option>
					{/each}
				</select>
				<Icon d={mdiMenuDown}></Icon>
			</div>
		</nav>
	</header>

	<main class="responsive">
		{@render children()}
	</main>

	<footer class="container">
		<p>
			{@html $_('powered_by', {
				values: {
					company: '<a href="https://moonsha.de" class="link" target="_blank">Moonshade</a>'
				}
			})} -
			<a href="https://github.com/Jigjick/Pasha-Anomaly" class="link" target="_blank"
				>{$_('source_code')}</a
			>
			-
			<a href="https://github.com/Jigjick/Pasha-Anomaly/issues/" class="link" target="_blank"
				>{$_('report_issue')}</a
			>
			-
			<a
				href="https://docs.google.com/forms/d/e/1FAIpQLSd6KcEn4Ay6aCS6a7cXvov9-kTDeHqODHcokpYTlrpB1p3Wbw/viewform?usp=header"
				class="link"
				target="_blank">{$_('announcement_designFeedback_footer')}</a
			>
		</p>
	</footer>

	<div class="snackbar" class:active={announcementActive}>
		<div class="max">{$_('announcement_designFeedback_text')}</div>
		<a
			class="inverse-link"
			target="_blank"
			href="https://docs.google.com/forms/d/e/1FAIpQLSd6KcEn4Ay6aCS6a7cXvov9-kTDeHqODHcokpYTlrpB1p3Wbw/viewform?usp=header"
		>
			{$_('announcement_designFeedback_link')}
		</a>
		<button class="transparent square" onclick={hideAnnouncement}><Icon d={mdiClose}></Icon></button>
	</div>
{/await}

<style>
	footer {
		text-align: center;
	}

	:global(.lang-select select),
	:global(.lang-select select:focus),
	:global(.lang-select option) {
		color: var(--on-surface) !important;
		background-color: var(--surface-container-high) !important;
		-webkit-text-fill-color: var(--on-surface) !important;
		opacity: 1 !important;
	}
</style>
