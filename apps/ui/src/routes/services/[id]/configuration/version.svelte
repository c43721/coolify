<script context="module" lang="ts">
	import type { Load } from '@sveltejs/kit';
	export const load: Load = async ({ fetch, params, url, stuff }) => {
		try {
			const { service } = stuff;
			if (service?.version && !url.searchParams.get('from')) {
				return {
					status: 302,
					redirect: `/services/${params.id}`
				};
			}
			const response = await get(`/services/${params.id}/configuration/version`);
			return {
				props: {
					...response
				}
			};
		} catch (error) {
			return {
				status: 500,
				error: new Error(`Could not load ${url}`)
			};
		}
	};
</script>

<script lang="ts">
	export let versions: any;
	export let type: any;
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { get, post } from '$lib/api';
	import { t } from '$lib/translations';
	import { errorNotification, supportedServiceTypesAndVersions } from '$lib/common';

	const { id } = $page.params;
	const from = $page.url.searchParams.get('from');

	let recommendedVersion = supportedServiceTypesAndVersions.find(
		({ name }) => name === type
	)?.recommendedVersion;
	async function handleSubmit(version: any) {
		try {
			await post(`/services/${id}/configuration/version`, { version });
			return await goto(from || `/services/${id}`);
		} catch (error) {
			return errorNotification(error);
		}
	}
</script>

<div class="flex space-x-1 p-6 font-bold">
	<div class="mr-4 text-2xl tracking-tight">{$t('forms.select_a_service_version')}</div>
</div>
{#if from}
	<div class="pb-10 text-center">
		Warning: you are about to change the version of this service.<br />This could cause problem
		after you restart the service,
		<span class="font-bold text-pink-600">like losing your data, incompatibility issues, etc</span
		>.<br />Only do if you know what you are doing.
	</div>
{/if}
<div class="flex flex-wrap justify-center">
	{#each versions as version}
		<div class="p-2">
			<form on:submit|preventDefault={() => handleSubmit(version)}>
				<button
					type="submit"
					class:bg-pink-500={recommendedVersion === version}
					class="box-selection relative flex text-xl font-bold hover:bg-pink-600"
					>{version}
					{#if recommendedVersion === version}
						<span class="absolute bottom-0 pb-2 text-xs">recommended</span>
					{/if}</button
				>
			</form>
		</div>
	{/each}
</div>
