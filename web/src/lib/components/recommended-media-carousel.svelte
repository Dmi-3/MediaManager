<script lang="ts">
	import emblaCarouselSvelte from 'embla-carousel-svelte';
	import { ChevronLeft, ChevronRight } from 'lucide-svelte';
	import AddMediaCard from '$lib/components/add-media-card.svelte';
	import { Skeleton } from '$lib/components/ui/skeleton';
	import type { MetaDataProviderSearchResult } from '$lib/api/api';

	let {
		media,
		isShow,
		isLoading
	}: {
		media: MetaDataProviderSearchResult[];
		isShow: boolean;
		isLoading: boolean;
	} = $props();

	let emblaApi: any = $state(undefined);
	let canScrollPrev = $state(false);
	let canScrollNext = $state(true);

	const emblaOptions = { align: 'start' as const, dragFree: true, containScroll: 'trimSnaps' as const };

	function onEmblaInit(event: CustomEvent) {
		emblaApi = event.detail;
		updateButtons();
		emblaApi.on('select', updateButtons);
		emblaApi.on('reInit', updateButtons);
	}

	function updateButtons() {
		if (!emblaApi) return;
		canScrollPrev = emblaApi.canScrollPrev();
		canScrollNext = emblaApi.canScrollNext();
	}

	function scrollPrev() { emblaApi?.scrollPrev(); }
	function scrollNext() { emblaApi?.scrollNext(); }
</script>

<div class="relative px-4">
	{#if !isLoading && media.length > 0}
		<button
			class="absolute -left-1 top-1/2 z-10 -translate-y-1/2 rounded-full bg-black/70 p-2 text-white shadow-lg backdrop-blur-sm transition-all hover:bg-black/90 disabled:pointer-events-none disabled:opacity-0"
			disabled={!canScrollPrev}
			onclick={scrollPrev}
			aria-label="Scroll left"
		>
			<ChevronLeft class="h-5 w-5" />
		</button>
		<button
			class="absolute -right-1 top-1/2 z-10 -translate-y-1/2 rounded-full bg-black/70 p-2 text-white shadow-lg backdrop-blur-sm transition-all hover:bg-black/90 disabled:pointer-events-none disabled:opacity-0"
			disabled={!canScrollNext}
			onclick={scrollNext}
			aria-label="Scroll right"
		>
			<ChevronRight class="h-5 w-5" />
		</button>
	{/if}

	<div
		class="overflow-hidden"
		use:emblaCarouselSvelte={{ options: emblaOptions }}
		onemblaInit={onEmblaInit}
	>
		<div class="flex gap-3">
			{#if isLoading}
				{#each { length: 8 } as _}
					<div class="aspect-[2/3] w-36 flex-none md:w-44">
						<Skeleton class="h-full w-full rounded-xl" />
					</div>
				{/each}
			{:else}
				{#each media as mediaItem (mediaItem.external_id)}
					<div class="w-36 flex-none md:w-44">
						<AddMediaCard {isShow} result={mediaItem} />
					</div>
				{/each}
			{/if}
		</div>
	</div>
</div>
