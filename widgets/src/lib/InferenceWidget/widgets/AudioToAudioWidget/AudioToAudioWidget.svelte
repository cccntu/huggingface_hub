<script>
	import type { WidgetProps } from "../../shared/types";

	import WidgetAudioTrack from "../../shared/WidgetAudioTrack/WidgetAudioTrack.svelte";
	import WidgetFileInput from "../../shared/WidgetFileInput/WidgetFileInput.svelte";
	import WidgetRadio from "../../shared/WidgetRadio/WidgetRadio.svelte";
	import WidgetRecorder from "../../shared/WidgetRecorder/WidgetRecorder.svelte";
	import WidgetSubmitBtn from "../../shared/WidgetSubmitBtn/WidgetSubmitBtn.svelte";
	import WidgetWrapper from "../../shared/WidgetWrapper/WidgetWrapper.svelte";
	import { getResponse } from "../../shared/helpers";

	export let apiToken: WidgetProps["apiToken"];
	export let apiUrl: WidgetProps["apiUrl"];
	export let model: WidgetProps["model"];
	export let noTitle: WidgetProps["noTitle"];

	let areSamplesVisible = true;
	let computeTime = "";
	let error: string = "";
	let file: Blob | File | null = null;
	let filename: string = "";
	let fileUrl: string;
	let isLoading = false;
	let isRecording = false;
	let modelLoading = {
		isLoading: false,
		estimatedTime: 0,
	};
	let output: AudioItem[] = [];
	let outputJson: string;
	let selectedSampleUrl = "";

	interface AudioItem {
		blob: string;
		label: string;
		src: string;
	}

	function onChangeRadio() {
		file = null;
		filename = "";
		fileUrl = "";
	}

	function onRecordStart() {
		areSamplesVisible = false;
		file = null;
		filename = "";
		fileUrl = "";
		isRecording = true;
	}

	function onSelectFile(updatedFile: Blob | File) {
		areSamplesVisible = false;
		isRecording = false;
		selectedSampleUrl = "";
		if (updatedFile.size !== 0) {
			const date = new Date();
			const time = date.toLocaleTimeString("en-US");
			filename =
				"name" in updatedFile
					? updatedFile.name
					: `Audio recorded from browser [${time}]`;
			file = updatedFile;
			fileUrl = URL.createObjectURL(file);
		}
	}

	async function getOutput(withModelLoading = false) {
		if (!file && !selectedSampleUrl) {
			error = "You must select or record an audio file";
			return;
		}
		const requestBody = file ? { file } : { url: selectedSampleUrl };
		isLoading = true;
		const res = await getResponse(
			apiUrl,
			model.modelId,
			requestBody,
			apiToken,
			parseOutput,
			withModelLoading
		);
		isLoading = false;
		// Reset values
		computeTime = "";
		error = "";
		modelLoading = { isLoading: false, estimatedTime: 0 };
		output = [];

		if (res.status === "success") {
			computeTime = res.computeTime;
			output = res.output;
			outputJson = res.outputJson;
		} else if (res.status === "loading-model") {
			modelLoading = {
				isLoading: true,
				estimatedTime: res.estimatedTime,
			};
			getOutput(true);
		} else if (res.status === "error") {
			error = res.error;
		}
	}

	function isValidOutput(arg: any): arg is AudioItem[] {
		return (
			Array.isArray(arg) &&
			arg.every(
				(x) =>
					typeof x.blob === "string" &&
					typeof x.label === "string" &&
					typeof x.src === "string"
			)
		);
	}

	function parseOutput(body: unknown): AudioItem[] {
		if (isValidOutput(body)) {
			for (const item of body) {
				item.src = `data:${item["content-type"]};base64,${item.blob}`;
			}
			return body;
		}
		return [];
	}
</script>

<WidgetWrapper
	{apiUrl}
	{computeTime}
	{error}
	{model}
	{modelLoading}
	{noTitle}
	{outputJson}
>
	<svelte:fragment slot="top">
		<form>
			<div class="flex items-center flex-wrap">
				<WidgetFileInput
					accept="audio/*"
					classNames="mt-1.5 mr-2"
					{onSelectFile}
				/>
				<span class="mt-1.5 mr-2">or</span>
				<WidgetRecorder
					classNames="mt-1.5"
					{onRecordStart}
					onRecordStop={onSelectFile}
				/>
			</div>
			{#if fileUrl}
				<WidgetAudioTrack classNames="mt-3" label={filename} src={fileUrl} />
			{/if}
			{#if model.widgetData}
				<details
					open={areSamplesVisible}
					class="text-gray-500 text-sm mt-4 mb-2"
				>
					<summary class="mb-2">Or pick a sample audio file</summary>
					<div class="mt-4 space-y-5">
						{#each model.widgetData as widgetData}
							<WidgetAudioTrack classNames="mt-3" src={String(widgetData.src)}>
								<WidgetRadio
									bind:group={selectedSampleUrl}
									classNames="mb-1.5"
									label={String(widgetData.label)}
									onChange={onChangeRadio}
									value={String(widgetData.src)}
								/>
							</WidgetAudioTrack>
						{/each}
					</div>
				</details>
			{/if}
			<WidgetSubmitBtn
				classNames="mt-2"
				isDisabled={isRecording}
				{isLoading}
				onClick={() => {
					getOutput();
				}}
			/>
		</form>
	</svelte:fragment>
	<svelte:fragment slot="bottom">
		{#each output as item}
			<div>{item.label}</div>
			:
			<WidgetAudioTrack classNames="mt-4" src={item.src} />
		{/each}
	</svelte:fragment>
</WidgetWrapper>
