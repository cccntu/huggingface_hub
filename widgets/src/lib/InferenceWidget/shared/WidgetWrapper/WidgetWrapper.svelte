<script>
	import type { WidgetProps, LoadingStatus } from "../types";

	import { onMount } from "svelte";
	import IconCross from "../../../Icons/IconCross.svelte";
	import WidgetFooter from "../WidgetFooter/WidgetFooter.svelte";
	import WidgetHeader from "../WidgetHeader/WidgetHeader.svelte";
	import WidgetInfo from "../WidgetInfo/WidgetInfo.svelte";
	import WidgetModelLoading from "../WidgetModelLoading/WidgetModelLoading.svelte";
	import { getModelStatus } from "../../shared/helpers";

	export let apiUrl: string;
	export let computeTime: string;
	export let error: string;
	export let model: WidgetProps["model"];
	export let modelLoading = {
		isLoading: false,
		estimatedTime: 0,
	};
	export let noTitle = false;
	export let outputJson: string;

	let isMaximized = false;
	let modelStatus: LoadingStatus = "unknown";

	onMount(() => {
		getModelStatus(apiUrl, model.modelId).then((status) => {
			modelStatus = status;
		});
	});

	function onClickMaximizeBtn() {
		isMaximized = !isMaximized;
	}
</script>

<div
	class="flex flex-col w-full max-w-full
	{isMaximized ? 'fixed inset-0 bg-white p-12 z-20' : ''}"
>
	{#if isMaximized}
		<button class="absolute top-6 right-12" on:click={onClickMaximizeBtn}>
			<IconCross classNames="text-xl text-gray-500 hover:text-black" />
		</button>
	{/if}
	<WidgetHeader {noTitle} pipeline={model.pipeline_tag}>
		{#if model.pipeline_tag === "fill-mask"}
			Mask token: <code>{model.mask_token}</code>
		{/if}
	</WidgetHeader>
	<slot name="top" />
	<WidgetInfo {computeTime} {error} {modelStatus} />
	{#if modelLoading.isLoading}
		<WidgetModelLoading estimatedTime={modelLoading.estimatedTime} />
	{/if}
	<slot name="bottom" />
	<WidgetFooter {onClickMaximizeBtn} {outputJson} />
</div>
