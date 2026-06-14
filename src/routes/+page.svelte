<script lang="ts">
	import type { PageData } from './$types';
	import { page } from '$app/state';

	let { data }: { data: PageData } = $props();

	const pageTitle = 'Abiee — Editorial Photography Gallery';
	const pageDescription =
		'Shapes in Light — An editorial curation of visual storytelling. Capturing brutalist horizons, delicate glass layers, and the profound depth of human stillness.';

	let ogImageUrl = $derived(
		`${page.url.origin}/og?title=${encodeURIComponent(pageTitle)}&description=${encodeURIComponent(pageDescription)}${data.photos.length > 0 ? `&image=${encodeURIComponent(data.photos[0].url)}` : ''}`
	);

	// Tag filtering
	type TagItem = PageData['tags'][number];
	type PhotoItem = PageData['photos'][number];
	type PhotoTagItem = PhotoItem['tags'][number];

	let activeTag = $derived(page.url.searchParams.get('tag'));
	let tagsWithPhotos = $derived(data.tags.filter((t: TagItem) => t.photoCount > 0));
	let filteredPhotos = $derived(
		activeTag
			? data.photos.filter((p: PhotoItem) =>
					p.tags.some((t: PhotoTagItem) => t.tag.slug === activeTag)
				)
			: data.photos
	);
	let activeTagName = $derived(
		activeTag ? tagsWithPhotos.find((t: TagItem) => t.slug === activeTag)?.name : null
	);
</script>

<svelte:head>
	<title>{pageTitle}</title>
	<meta name="description" content={pageDescription} />
	<meta property="og:title" content={pageTitle} />
	<meta property="og:description" content={pageDescription} />
	<meta property="og:image" content={ogImageUrl} />
	<meta property="og:image:width" content="1200" />
	<meta property="og:image:height" content="630" />
	<meta name="twitter:title" content={pageTitle} />
	<meta name="twitter:description" content={pageDescription} />
	<meta name="twitter:image" content={ogImageUrl} />
	<meta name="twitter:card" content="summary_large_image" />

	<!-- Structured Data: ImageGallery -->
	{@html `<script type="application/ld+json">${JSON.stringify({
		'@context': 'https://schema.org',
		'@type': 'ImageGallery',
		name: 'Lensy — Shapes in Light',
		description: pageDescription,
		image: data.photos.slice(0, 5).map((p: PageData['photos'][number]) => ({
			'@type': 'ImageObject',
			contentUrl: p.url,
			name: p.title || 'Untitled',
			description:
				p.description ||
				`${p.make || ''} ${p.model || ''} · ${p.focalLength}mm ${p.aperture}`.trim()
		}))
	})}</script>`}
</svelte:head>

<div
	class="min-h-dvh overflow-x-hidden bg-white text-[#555] antialiased selection:bg-black/10 martab-layout"
>
	<main class="relative z-10 mx-auto max-w-[1600px] px-6 py-16 md:px-12 lg:px-16 lg:py-24">
		<!-- Centered Brand & Tag Navigation -->
		<header class="mb-16 flex flex-col items-center gap-6 text-center">
			<a
				href="/"
				class="text-3xl font-normal tracking-[0.25em] uppercase text-[#111] hover:opacity-85 transition-opacity"
			>
				Abiee
			</a>

			{#if tagsWithPhotos.length > 0}
				<nav class="flex flex-wrap items-center justify-center gap-x-8 gap-y-3 text-[11px] tracking-[0.15em] uppercase" aria-label="Filter by tag">
					<a
						href="/"
						class="transition-all {!activeTag ? 'active-tag font-bold' : 'inactive-tag font-normal'}"
					>
						All
					</a>
					{#each tagsWithPhotos as tag}
						<a
							href="/?tag={tag.slug}"
							class="transition-all {activeTag === tag.slug ? 'active-tag font-bold' : 'inactive-tag font-normal'}"
						>
							{tag.name}
						</a>
					{/each}
				</nav>
			{/if}
		</header>

		<!-- Active filter indicator -->
		{#if activeTagName}
			<div class="mb-12 flex items-center justify-center gap-3">
				<h2 class="text-xs font-mono tracking-widest text-[#999] uppercase">{activeTagName}</h2>
				<span class="text-[10px] font-mono text-black/30">({filteredPhotos.length})</span>
			</div>
		{/if}

		<!-- Clean Multi-Column Masonry Grid -->
		<section
			class="columns-1 gap-6 sm:columns-2 lg:columns-3 xl:columns-4"
			aria-label="Photography collection"
		>
			{#each filteredPhotos as photo (photo.id)}
				<div class="martab-grid-item">
					<a href="/photo/{photo.id}" class="block bg-black/[0.01]">
						<img
							src={photo.url}
							alt={photo.title || 'Photography'}
							loading="lazy"
						/>
					</a>
				</div>
			{/each}
		</section>
	</main>
</div>

<style>
	.martab-layout {
		font-family: 'Courier New', Courier, monospace;
	}

	.martab-layout a {
		font-family: 'Courier New', Courier, monospace;
	}

	.martab-layout a.active-tag {
		color: #111111;
	}

	.martab-layout a.inactive-tag {
		color: #111111;
		opacity: 0.4;
	}

	.martab-layout a.inactive-tag:hover {
		opacity: 0.85;
	}

	.martab-grid-item {
		margin-bottom: 1.5rem;
		break-inside: avoid;
		opacity: 0;
		animation: fade-in 0.8s cubic-bezier(0.32, 0.72, 0, 1) forwards;
	}

	.martab-grid-item img {
		width: 100%;
		height: auto;
		display: block;
		transition: opacity 0.3s ease;
	}

	.martab-grid-item img:hover {
		opacity: 0.8;
	}

	@keyframes fade-in {
		0% {
			opacity: 0;
		}
		100% {
			opacity: 1;
		}
	}
</style>
