<script>
	export let videoId;
	export let playLabel = 'Play';
	export let params = '';
	export let posterSrc = '';
	export let posterSrcSet = '';
	export let posterWidth = 560;
	export let posterHeight = 315;

	let activated = false;
	let hovered = false;

	$: videoId, (activated = false);
	$: computedParams = (() => {
		const p = new URLSearchParams(params);
		p.append('autoplay', '1');
		p.append('mute', '1');
		p.append('modestbranding', '1');
		// p.append('playsinline', '1');
		// p.append('enablejsapi', '1');
		p.append('rel', '0');
		p.append('showinfo', '0');

		return p.toString();
	})();

	function focus(node) {
		node.focus();
	}
</script>

<svelte:head>
	<link rel="preconnect" href="https://i.ytimg.com" />
	{#if hovered}
		<!-- <link rel="preconnect" href="https://www.youtube.com" /> -->
		<link rel="preconnect" href="https://www.youtube-nocookie.com" />
		<link rel="preconnect" href="https://www.google.com" />
	{/if}
</svelte:head>

<div
	class='lite-youtube'
	class:lite-youtube-activated={activated}
	on:pointerover|once={() => (hovered = true)}
	on:click={() => (activated = true)}
	on:keypress={() => (activated = true)}
	role="button"
	tabindex="0"
>
	{#key videoId}
		<picture>
			<img class="lite-youtube-poster" alt={playLabel} src={posterSrc} srcset={posterSrcSet} />
		</picture>
	{/key}
	<slot>
		<button type="button" class="lite-playbtn" aria-label={playLabel} />
	</slot>
	{#if activated}
		<iframe
			title={playLabel}
			width={posterWidth}
			height={posterHeight}
			allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
			allowfullscreen
			src="https://www.youtube-nocookie.com/embed/{encodeURIComponent(videoId)}?{computedParams}"
			use:focus
		/>
	{/if}
</div>

<style global>
	:global(.lite-youtube) {
		background-color: #000000;
		position: relative;
		display: block;
		contain: content;
		cursor: pointer;
	}
	/* gradient */
	:global(.lite-youtube::before) {
		content: '';
		display: block;
		position: absolute;
		top: 0;
		background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAADGCAYAAAAT+OqFAAAAdklEQVQoz42QQQ7AIAgEF/T/D+kbq/RWAlnQyyazA4aoAB4FsBSA/bFjuF1EOL7VbrIrBuusmrt4ZZORfb6ehbWdnRHEIiITaEUKa5EJqUakRSaEYBJSCY2dEstQY7AuxahwXFrvZmWl2rh4JZ07z9dLtesfNj5q0FU3A5ObbwAAAABJRU5ErkJggg==);
		background-position: top;
		background-repeat: repeat-x;
		height: 60px;
		padding-bottom: 50px;
		width: 100%;
		transition: all 0.2s cubic-bezier(0, 0, 0.2, 1);
		box-sizing: unset;
		z-index: 1;
	}
	/* responsive iframe with a 16:9 aspect ratio
    thanks https://css-tricks.com/responsive-iframes/
	*/
	:global(.lite-youtube::after) {
		content: '';
		display: block;
		padding-bottom: calc(100% / (16 / 9));
	}
	:global(.lite-youtube) > :global(iframe) {
		width: 100%;
		height: 100%;
		position: absolute;
		top: 0;
		left: 0;
		border: 0;
	}
	/* poster */
	:global(.lite-youtube-poster) {
		width: 100%;
		height: 100%;
		position: absolute;
		top: 0;
		left: 0;
		object-fit: cover;
	}
	/* play button */
	:global(.lite-youtube) > :global(.lite-playbtn) {
		width: 68px;
		height: 48px;
		position: absolute;
		cursor: pointer;
		transform: translate3d(-50%, -50%, 0);
		top: 50%;
		left: 50%;
		z-index: 1;
		background-color: transparent;
		/* YT's actual play button svg */
		background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 68 48"><path fill="%23f00" fill-opacity="0.8" d="M66.52,7.74c-0.78-2.93-2.49-5.41-5.42-6.19C55.79,.13,34,0,34,0S12.21,.13,6.9,1.55 C3.97,2.33,2.27,4.81,1.48,7.74C0.06,13.05,0,24,0,24s0.06,10.95,1.48,16.26c0.78,2.93,2.49,5.41,5.42,6.19 C12.21,47.87,34,48,34,48s21.79-0.13,27.1-1.55c2.93-0.78,4.64-3.26,5.42-6.19C67.94,34.95,68,24,68,24S67.94,13.05,66.52,7.74z"></path><path d="M 45,24 27,14 27,34" fill="%23fff"></path></svg>');
		filter: grayscale(100%);
		transition: filter 0.1s cubic-bezier(0, 0, 0.2, 1);
		border: none;
		outline: 0;
	}
	:global(.lite-youtube:hover) > :global(.lite-playbtn),
	:global(.lite-youtube) :global(.lite-playbtn:focus) {
		filter: none;
	}
	/* Post-click styles */
	:global(.lite-youtube.lite-youtube-activated) {
		cursor: unset;
	}
	:global(.lite-youtube.lite-youtube-activated::before),
	:global(.lite-youtube.lite-youtube-activated) > :global(.lite-playbtn) {
		opacity: 0;
		pointer-events: none;
	}
</style>