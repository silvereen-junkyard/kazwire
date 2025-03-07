<script lang="ts" context="module">
	declare var __uv$config: any;
</script>

<script lang="ts">
	import type { PageData } from './$types';
	export let data: PageData;

	import { PUBLIC_API_BASE_URL } from '$env/static/public';

	import { onMount } from 'svelte';

	// Turns a search into a valid URL
	function search(input: string) {
		let template: string = 'https://www.google.com/search?q=%s&hl=en';
		try {
			// input is a valid URL:
			// eg: https://example.com, https://example.com/test?q=param
			return new URL(input).toString();
		} catch (err) {
			// input was not a valid URL
		}

		try {
			// input is a valid URL when http:// is added to the start:
			// eg: example.com, https://example.com/test?q=param
			const url: URL = new URL(`http://${input}`);
			// only if the hostname has a TLD/subdomain
			if (url.hostname.includes('.')) return url.toString();
		} catch (err) {
			// input was not valid URL
		}

		// input may have been a valid URL, however the hostname was invalid

		// Attempts to convert the input to a fully qualified URL have failed
		// Treat the input as a search query
		return template.replace('%s', encodeURIComponent(input));
	}

	function encodeURL(url: string): string {
		// check if the service worker is installed
		navigator.serviceWorker.getRegistrations().then((registrations) => {
			if (registrations.length === 0) {
				// Service worker is not installed so register it
				registerServiceWorker();
			}
		});

		return __uv$config.prefix + __uv$config.encodeUrl(search(url));
	}

	function registerServiceWorker() {
		// Register the service worker
		navigator.serviceWorker.register('/uv.js', { scope: __uv$config.prefix }).then((reg) => {
			if (reg.installing) {
				const sw = reg.installing || reg.waiting;
				sw.onstatechange = function () {
					if (sw.state === 'installed') {
						// Reload page
						window.location.reload();
					}
				};
			}
		});
	}

	import { experiments } from '$lib/experiments';
	let showSuggestedApps: boolean = false;

	import SmallBox from '$lib/components/Box/SmallBox.svelte';
	onMount(() => {
		registerServiceWorker();

		// Check if the app is liked and set the isLiked store
		if (appLike.isLiked(data.app.id)) {
			isLiked.set(true);
		} else {
			isLiked.set(false);
		}
	});

	// Fullscreen the iframe
	function fullScreen() {
		const iframe: HTMLIFrameElement = document.getElementById('iframe') as HTMLIFrameElement;
		iframe.requestFullscreen();
	}

	let expanded: boolean = false;
	// Expand the iframe to fill the screen
	function expandiFrame(): void {
		if (!loadedFrame) {
			return;
		}

		const document: Document = window.document;
		const frame: HTMLIFrameElement = document.getElementById('iframe') as HTMLIFrameElement;

		document.body.style.overflow = 'hidden';
		// Settings required for the frame to fill the screen
		frame.style.position = 'fixed';
		frame.style.top = '0px';
		frame.style.bottom = '0px';
		frame.style.left = '0px';
		frame.style.right = '0px';
		frame.style.height = '100%';
		frame.style.width = '100%';
		frame.style.zIndex = '500';
		frame.style.border = 'none';

		frame.classList.toggle('rounded-t-lg');

		expanded = true;
	}

	function shrinkiFrame() {
		const document: Document = window.document;
		const frame: HTMLIFrameElement = document.getElementById('iframe') as HTMLIFrameElement;

		document.body.style.overflow = 'auto';
		// Settings required for the frame to fill the screen
		frame.style.position = 'relative';
		frame.style.top = '0px';
		frame.style.bottom = '0px';
		frame.style.left = '0px';
		frame.style.right = '0px';
		frame.style.height = '100%';
		frame.style.width = '100%';
		frame.style.zIndex = '9999';
		frame.style.border = 'none';

		frame.classList.toggle('rounded-t-lg');

		expanded = false;
	}

	import { appLike } from '$lib/likeContent';
	import { isLiked } from '$lib/stores';

	import Icon from '@iconify/svelte';
	import Tag from '$lib/components/Tag.svelte';

	import Vert from '$lib/components/Google/Vert.svelte';
	import Vert2 from '$lib/components/Google/Vert2.svelte';
	import Leaderboard from '$lib/components/Google/Leaderboard.svelte';
	import Horz from '$lib/components/Google/Horz.svelte';

	import { isLoading, _ } from 'svelte-i18n';
	let innerWidth: number = 0;

	let loadedFrame: boolean = false;
	let loadingApp: boolean = false;
	async function loadFrame() {
		loadedFrame = true;
		loadingApp = true;
		// Just in case wait 5 seconds before removing the loading screen
		setTimeout(() => {
			loadedApp();
		}, 5000);
	}

	function addView() {
		// Add to the views
		fetch(PUBLIC_API_BASE_URL + '/api/apps/' + data.app.id + '/views', {
			method: 'POST'
		});
	}

	function loadedApp() {
		// Wait 0.5 second before removing the loading screen
		setTimeout(() => {
			// Remove the hidden class from the iframe
			const iframe: HTMLIFrameElement = document.getElementById('iframe') as HTMLIFrameElement;
			iframe.classList.remove('opacity-0');

			loadingApp = false;
		}, 500);
	}

	function moveShrinkButton(e: MouseEvent) {
		// Wait 2 seconds before moving the button
		console.log(e);
		setTimeout(() => {
			// grab the current cursor position

			// Check if the button is still being hovered over by grabbing the event x, y and comparing it to the current x, y
			if (e.x == mousePos.x && e.y == mousePos.y) {
				// Move the button to the bottom right
				const button: HTMLButtonElement = document.getElementById(
					'shrinkButton'
				) as HTMLButtonElement;
				button.classList.remove('m-4');
				button.classList.add('ml-16');
				button.classList.add('mt-4');

				// Return it to the top left after 4 seconds
				setTimeout(() => {
					button.classList.remove('ml-16');
					button.classList.remove('mt-4');
					button.classList.add('m-4');
				}, 4000);
			}
		}, 2000);
	}

	let mousePos = { x: 0, y: 0 };
</script>

<svelte:window bind:innerWidth on:mousemove={(e) => (mousePos = { x: e.x, y: e.y })} />
<svelte:head>
	<title>{data.app.name} - Use Unblocked on Kazwire!</title>
	<meta property="og:title" content={data.app.name} />
	<meta name="description" content={data.app.description} />
	<meta property="og:description" content="Play {data.app.name} for free now on Kazwire!" />
	<meta property="og:image" content="/app/img/{data.app.image}" />

	<script src="/uv/uv.bundle.js" async={false}></script>
	<script src="/uv/uv.config.js" async={false}></script>
	<script src="/uv.js" async={false}></script>
</svelte:head>

{#if expanded}
	<!-- Button to shrink the iframe -->
	<!-- if the user hovers over the button for two seconds move it so that they can access stuff below it send the event every 0.1 seconds -->
	<button
		id="shrinkButton"
		class="absolute left-0 top-0 z-[5000] m-4 rounded-full bg-secondary p-2 opacity-40"
		on:click={() => shrinkiFrame()}
		on:mouseover={(e) => moveShrinkButton(e)}
	>
		<Icon class="h-6 w-6 text-white" icon="ic:round-compress" />
	</button>
{/if}

{#if !$isLoading}
	<div class="relative flex flex-row justify-center">
		<div class="float-left flex h-fit pb-5 sm:w-full md:w-[820px] lg:w-[1000px] xl:w-full">
			{#if innerWidth > 1224}
				<Vert />
			{/if}
			<div class="align-center mb-14 h-[calc(80vh-200px)] min-h-[24rem] flex-grow">
				<div id="frame" class="h-full w-full rounded-t-lg bg-white">
					{#if !loadedFrame}
						<div class="relative flex h-full items-center justify-center overflow-hidden">
							<img
								class="absolute z-20 h-full w-full object-cover opacity-60 blur-lg"
								src="/app/img/{data.app.image}"
								alt="App"
							/>
							<div class="absolute z-10 h-full w-full rounded-t-lg bg-black" />

							<!-- Content on top of the image -->
							<div class="absolute z-30 flex flex-col items-center justify-center">
								<h1
									class="text-center text-3xl font-bold text-white sm:text-5xl md:text-5xl lg:text-8xl"
								>
									{data.app.name}
								</h1>

								<!-- Access now button -->
								<button
									class="lg:btn-xl btn mt-8"
									on:click={() => addView()}
									on:click={() => loadFrame()}
								>
									{$_('pages.apps.access_now')}
									<Icon icon="carbon:play-filled" class="my-auto ml-1 inline-block" />
								</button>
							</div>
						</div>
					{:else}
						{#if loadingApp}
							<!-- Loading animation -->
							<div
								class="relative flex h-full items-center justify-center rounded-t-lg bg-black transition-all"
							>
								<div class="absolute z-30 flex flex-col items-center justify-center gap-8">
									<div class="flex flex-col items-center gap-8 sm:flex-row">
										<img src="/logo.png" alt="Loading" class="h-16 w-16" />
										<h1
											class="text-center text-3xl font-bold text-white sm:text-5xl md:text-5xl lg:text-8xl"
										>
											Kazwire
										</h1>
									</div>
									<Icon icon="line-md:loading-alt-loop" class="animate-spin text-6xl text-white" />
								</div>
							</div>
						{/if}
						<!-- Static app -->
						{#if data.app.embedURL == null}
							<iframe
								src={'/app/static/' + data.app.id + '/index.html'}
								class="h-full w-full rounded-t-lg bg-white opacity-0"
								id="iframe"
								title={data.app.name}
								on:load={() => loadedApp()}
							/>
							<!-- Ruffle app -->
						{:else if data.app.embedURL != null}
							<iframe
								class="h-full w-full rounded-t-lg bg-white opacity-0"
								id="iframe"
								title={data.app.name}
								src={encodeURL(data.app.embedURL)}
								on:load={() => loadedApp()}
							/>
						{/if}
					{/if}
				</div>

				<div
					class="relative mt-2 w-full items-center rounded-b-lg bg-tertiary text-black dark:bg-tertiaryDark dark:text-white"
				>
					<div class="float-right mr-5">
						<button class="mt-4 fill-white" on:click={() => fullScreen()}>
							<!-- Full screen -->
							<Icon class="h-6 w-6" icon="ic:baseline-fullscreen" />
						</button>
					</div>
					<div class="float-right mr-5">
						<button class="mt-4" on:click={() => expandiFrame()}>
							<!-- Fill screen -->
							<Icon class="h-6 w-6" icon="ic:round-expand" />
						</button>
					</div>
					<div class="float-right mr-5">
						<button id="heart" class="mt-4" on:click={() => appLike.toggle(data.app.id)}>
							<!-- Heart -->
							{#if $isLiked}
								<Icon class="h-6 w-6 text-red-500" icon="mdi:heart" />
							{:else}
								<Icon class="h-6 w-6" icon="mdi:heart-outline" />
							{/if}
						</button>
					</div>
					<div class="flex">
						<!-- Logo -->
						<img src="/logo.png" alt="Logo" class="my-auto ml-4 h-6 w-6" />
						<!-- Name -->
						<div class="ml-2 truncate text-2xl font-bold leading-[3.5rem]">
							{data.app.name}
						</div>
					</div>
				</div>
			</div>
		</div>
		{#if innerWidth > 824}
			<Vert2 />
		{/if}
	</div>

	{#if innerWidth > 730}
		<Leaderboard />
	{:else}
		<Horz />
	{/if}

	<!-- Bottom area for displaying more information about the app -->
	<div
		class="rounded-lg bg-tertiary p-5 align-middle text-black dark:bg-tertiaryDark dark:text-white"
	>
		<h1 class="text-3xl font-bold">{data.app.name}</h1>
		<p class="text-gray-800 dark:text-gray-200">
			{data.app.developer}
		</p>
		<p class="mt-1">
			{data.app.description}
		</p>
		<!-- Line -->
		<div class="my-2 h-[2px] w-10 rounded-lg bg-gray-400 dark:bg-gray-700" />
		<div class="flex">
			<p class="text-gray-600 dark:text-gray-300">
				{data.app.views} View{#if data.app.views != 1}s{/if}
			</p>
		</div>
	</div>
{/if}
