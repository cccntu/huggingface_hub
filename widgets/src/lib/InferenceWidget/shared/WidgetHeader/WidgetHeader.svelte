<script>
	import IconInfo from "../../../Icons/IconInfo.svelte";
	import IconLightning from "../../../Icons/IconLightning.svelte";
	import ModelPipelineTag from "../../../ModelPipelineTag/ModelPipelineTag.svelte";

	export let noTitle = false;
	export let pipeline: string | undefined;
</script>

<div class="font-semibold flex items-center mb-2">
	{#if !noTitle}
		<div class="text-lg flex items-center">
			<IconLightning classNames="-ml-1 mr-1 text-yellow-500" />
			Hosted inference API
		</div>
		<a target="_blank" href="/docs">
			<IconInfo classNames="ml-1.5 text-sm text-gray-400 hover:text-black" />
		</a>
	{/if}
</div>
<div class="flex items-center text-sm text-gray-500 mb-1.5">
	{#if pipeline}
		<ModelPipelineTag {pipeline} />
	{/if}
	<div class="ml-auto">
		<slot />
	</div>
</div>
