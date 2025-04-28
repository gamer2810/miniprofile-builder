<script lang="ts">
	import { browser } from '$app/environment';
	import { onMount } from 'svelte';

	// --- Configuration ---
	// Base URL for the embeddable content
	let baseUrl = 'https://gamer2810.github.io/steam-miniprofile/';

	// Configuration Variables
	let accountId: string = ''; // Required
	let lang: string = 'english'; // Default language
	let appId: string = ''; // Optional
	let interactive: boolean = false; // Optional, default false
	let vanityId: string = ''; // Required if interactive is true

	// List of available languages for the dropdown
	const availableLanguages = [
		'bulgarian', 'danish', 'english', 'french', 'greek', 'italian', 'koreana',
		'polish', 'brazilian', 'russian', 'latam', 'swedish', 'tchinese', 'ukrainian',
		'czech', 'dutch', 'finnish', 'german', 'hungarian', 'japanese', 'norwegian',
		'portuguese', 'romanian', 'schinese', 'spanish', 'thai', 'turkish', 'vietnamese'
	];

	let currentEmbedUrl: string = ''; // To display the generated URL

	// --- Reactive URL Generation ---
	$: iframeSrc = (() => {
		try {
			// Start with the base URL
			const url = new URL(baseUrl);

			// Add required accountId
			if (accountId) {
				url.searchParams.set('accountId', accountId);
			} else {
				// If no accountId, return a placeholder or base URL to avoid errors
				currentEmbedUrl = baseUrl; // Update display URL to base if ID missing
				return baseUrl; // Don't proceed if required ID is missing
			}

			// Add optional parameters if they have values
			if (lang && lang !== 'english') { // Only add if not default
				url.searchParams.set('lang', lang);
			}
			if (appId) {
				url.searchParams.set('appId', appId);
			}
			if (interactive) {
				url.searchParams.set('interactive', 'true');
				if (vanityId) { // vanityId is needed when interactive is true
					url.searchParams.set('vanityId', vanityId);
				} else {
                    console.warn("Interactive mode enabled but Vanity ID is missing.");
                }
			}

			currentEmbedUrl = url.toString(); // Update the display URL
			return currentEmbedUrl; // Return the complete URL string
		} catch (e) {
			console.error('Error constructing URL:', e);
			// Return a fallback or the base URL if construction fails
			currentEmbedUrl = baseUrl; // Update display URL
			return baseUrl;
		}
	})();

	// --- Clipboard Functionality ---
	let copyButtonText = 'Copy';
	function copyUrlToClipboard() {
		if (navigator.clipboard) {
			navigator.clipboard.writeText(currentEmbedUrl)
				.then(() => {
					copyButtonText = 'Copied!';
					setTimeout(() => { copyButtonText = 'Copy'; }, 2000); // Reset after 2 seconds
				})
				.catch(err => {
					console.error('Failed to copy URL: ', err);
					copyButtonText = 'Error';
					setTimeout(() => { copyButtonText = 'Copy'; }, 2000);
				});
		} else {
			console.warn('Clipboard API not available.');
            copyButtonText = 'Manual Copy';
		}
	}

	onMount(() => {
		// No specific onMount logic needed for this version
	});

</script>

<div class="container mx-auto p-4 lg:p-8 space-y-6">

    <header class="text-center">
        <h1 class="h1 font-bold">Steam Mini-Profile Embed Builder</h1>
        <p class="text-lg text-surface-600 dark:text-surface-400 mt-1">
            Configure and generate an embeddable Steam profile card.
        </p>
        <nav class="text-sm text-surface-500 dark:text-surface-500 mt-2 space-x-3">
            <span>Source:</span>
            <a href="https://github.com/gamer2810/steam-miniprofile" target="_blank" rel="noopener noreferrer" class="anchor">GitHub Repo</a>
            <span>|</span>
             <a href="https://ark.k3k.dev" target="_blank" rel="noopener noreferrer" class="anchor">Developer Blog (ark.k3k.dev)</a>
        </nav>
    </header>

    <hr class="opacity-50"/>

	<div class="grid grid-cols-1 lg:grid-cols-2 gap-8">

		<div class="flex flex-col space-y-6">
			<h3 class="h3">Configurations</h3>

			<label class="label">
				<span>Steam Account ID (Required)</span>
				<input class="input" type="text" placeholder="e.g., 7656119..." bind:value={accountId} required />
				<small>Find yours at <a class="anchor-font-color" href="https://steamdb.info/calculator/">https://steamdb.info/calculator/</a></small>
                {#if !accountId && currentEmbedUrl !== baseUrl}
                    <small class="text-error-500">Account ID is required for the embed to work.</small>
                {/if}
			</label>

            <label class="label">
                <span>Language</span>
                <select class="select" bind:value={lang}>
                    {#each availableLanguages as language}
                        <option value={language}>{language.charAt(0).toUpperCase() + language.slice(1)}</option>
                    {/each}
                </select>
            </label>

            <label class="label">
				<span>Favorite Game App ID (Optional)</span>
				<input class="input" type="text" placeholder="e.g., 730 (CS:GO)" bind:value={appId} />
                <small>Displays playtime for a specific game.</small>
				<br>
				<small>Find Game's App ID at <a class="anchor-font-color" href="https://steamdb.info/search/">https://steamdb.info/search/</a></small>
			</label>

            <div class="flex items-center space-x-2 cursor-pointer p-2 rounded-container-token hover:bg-surface-100 dark:hover:bg-surface-800" on:click={() => interactive = !interactive} on:keypress>
                <input type="checkbox" class="checkbox" id="interactive-checkbox" bind:checked={interactive} />
                <label for="interactive-checkbox" class="cursor-pointer">Make profile clickable?</label>
            </div>


            {#if interactive}
                <label class="label">
                    <span>Steam Vanity ID (Required if clickable)</span>
                    <input class="input" type="text" placeholder="e.g., your_custom_url_name" bind:value={vanityId} required />
                    <small>The part after /id/ in your profile URL (e.g., https://steamcommunity.com/id/<b>your_custom_url_name</b>).</small>
                     {#if !vanityId}
                        <small class="text-error-500">Vanity ID is required when 'clickable' is enabled.</small>
                    {/if}
                </label>
            {/if}

		</div>

		<div class="flex flex-col space-y-6">
			<div>
				<h3 class="h3 mb-2">Embed URL</h3>
				<div class="input-group">
                    <input type="text" class="input flex-grow" value={currentEmbedUrl} readonly aria-label="Generated Embed URL"/>
					<button class="btn variant-filled-primary" on:click={copyUrlToClipboard} disabled={!accountId}>{copyButtonText}</button> 
				</div>
			</div>

			<h3 class="h3 mb-4">Preview</h3>
			<div class="flex-grow flex flex-col">
				<div class="relative w-full" style="padding-bottom: 52.5%;"> 
					{#key iframeSrc} 
						{#if accountId}
                            <iframe
                                title="Steam Mini Profile Preview"
                                src={iframeSrc}
								allowtransparency="true"
								class="absolute top-0 left-0 w-full h-full border-0 rounded"
                                loading="lazy"
                                name="steamMiniProfilePreview"
                                scrolling="no"
                                frameborder="0"
                                allowfullscreen="false"
                            ></iframe>
                        {:else}
                             <div class="absolute top-0 left-0 w-full h-full flex items-center justify-center bg-surface-100 dark:bg-surface-800 rounded">
                                <p class="text-center text-surface-500 px-4">Please enter a Steam Account ID above to generate the preview.</p>
                            </div>
                        {/if}
					{/key}
				</div>
			</div>
		</div>
	</div>

    <footer class="text-center text-sm text-surface-500 dark:text-surface-500 mt-8 pt-4 border-t border-surface-300/30 dark:border-surface-700/30">
        <p>Steam Mini-Profile Embed Builder | Created based on <a href="https://github.com/gamer2810/steam-miniprofile" target="_blank" rel="noopener noreferrer" class="anchor">gamer2810/steam-miniprofile</a></p>
        <p>Check out the <a href="https://ark.k3k.dev" target="_blank" rel="noopener noreferrer" class="anchor">developer's blog</a>.</p>
    </footer>

</div>