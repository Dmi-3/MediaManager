<script lang="ts">
	import { Star, ImageOff, LoaderCircle, Check } from 'lucide-svelte';
	import { goto } from '$app/navigation';
	import { resolve } from '$app/paths';
	import type { MetaDataProviderSearchResult } from '$lib/api/api';
	import client from '$lib/api';

	let loading = $state(false);
	let { result, isShow = true }: { result: MetaDataProviderSearchResult; isShow: boolean } =
		$props();

	async function addMedia() {
		if (result.added || loading) return;
		loading = true;
		let data;
		if (isShow) {
			const response = await client.POST('/api/v1/tv/shows', {
				params: {
					query: {
						show_id: result.external_id,
						metadata_provider: result.metadata_provider as 'tmdb' | 'tvdb',
						language: result.original_language ?? undefined
					}
				}
			});
			data = response.data;
		} else {
			const response = await client.POST('/api/v1/movies', {
				params: {
					query: {
						movie_id: result.external_id,
						metadata_provider: result.metadata_provider as 'tmdb' | 'tvdb',
						language: result.original_language ?? undefined
					}
				}
			});
			data = response.data;
		}
		if (isShow) {
			await goto(resolve('/dashboard/tv/[showId]', { showId: data?.id ?? '' }), { invalidateAll: true });
		} else {
			await goto(resolve('/dashboard/movies/[movieId]', { movieId: data?.id ?? '' }), { invalidateAll: true });
		}
		loading = false;
	}

	function goToExisting() {
		if (isShow) {
			goto(resolve('/dashboard/tv/[showId]', { showId: result.id ?? '' }), { invalidateAll: true });
		} else {
			goto(resolve('/dashboard/movies/[movieId]', { movieId: result.id ?? '' }), { invalidateAll: true });
		}
	}
</script>

<div
	class="group relative aspect-[2/3] w-full cursor-pointer overflow-hidden rounded-xl shadow-lg"
	onclick={result.added ? goToExisting : addMedia}
	role="button"
	tabindex="0"
	onkeydown={(e) => e.key === 'Enter' && (result.added ? goToExisting() : addMedia())}
>
	{#if result.poster_path != null}
		<img
			class="h-full w-full object-cover transition-transform duration-500 group-hover:scale-105"
			src={result.poster_path}
			alt="{result.name} poster"
		/>
	{:else}
		<div class="flex h-full w-full items-center justify-center bg-gray-800">
			<ImageOff class="h-12 w-12 text-gray-600" />
		</div>
	{/if}

	{#if result.added}
		<div class="absolute left-2 top-2 flex items-center gap-1 rounded-full bg-green-500/90 px-2 py-0.5 text-xs font-medium text-white backdrop-blur-sm">
			<Check class="h-3 w-3" />
			Available
		</div>
	{/if}

	<div class="absolute inset-x-0 bottom-0 bg-gradient-to-t from-black/95 via-black/50 to-transparent p-3 pt-12 transition-opacity duration-200 group-hover:opacity-0">
		<h3 class="truncate text-sm font-semibold leading-tight text-white">{result.name}</h3>
		{#if result.year != null}
			<span class="text-xs text-gray-400">{result.year}</span>
		{/if}
	</div>

	<div class="absolute inset-0 flex flex-col justify-end bg-black/85 p-3 opacity-0 transition-opacity duration-200 group-hover:opacity-100">
		{#if result.vote_average != null}
			<div class="mb-2 flex items-center gap-1">
				<Star class="h-3 w-3 fill-amber-400 text-amber-400" />
				<span class="text-xs text-amber-400">{Math.round(result.vote_average * 10) / 10}/10</span>
			</div>
		{/if}
		<h3 class="mb-1 text-sm font-bold leading-tight text-white">
			{result.name}
			{#if result.year != null}
				<span class="font-normal text-gray-400">({result.year})</span>
			{/if}
		</h3>
		{#if result.overview && result.overview !== ''}
			<p class="mb-3 line-clamp-4 text-xs leading-relaxed text-gray-300">{result.overview}</p>
		{/if}
		{#if result.added}
			<button
				class="w-full rounded-lg bg-gray-600 py-2 text-sm font-semibold text-white transition-colors hover:bg-gray-500"
				onclick|stopPropagation={goToExisting}
			>
				View {isShow ? 'Show' : 'Movie'}
			</button>
		{:else}
			<button
				class="flex w-full items-center justify-center gap-2 rounded-lg bg-amber-500 py-2 text-sm font-semibold text-black transition-colors hover:bg-amber-400 disabled:opacity-60"
				disabled={loading}
				onclick|stopPropagation={addMedia}
			>
				{#if loading}
					<LoaderCircle class="h-4 w-4 animate-spin" />
					Adding...
				{:else}
					Add {isShow ? 'Show' : 'Movie'}
				{/if}
			</button>
		{/if}
	</div>
</div>
