<script>
	import IconSpin from "../../../Icons/IconSpin.svelte";

	export let accept = "image/*";
	export let isLoading = false;
	export let label =
		"Drag image file here or click to browse from your computer";
	export let onSelectFile: (file: File) => void;

	let fileInput: HTMLInputElement;
	let isDragging = false;
	let imgSrc = "";

	function onChange() {
		const file = fileInput.files?.[0];
		if (file) {
			imgSrc = URL.createObjectURL(file);
			onSelectFile(file);
		}
	}
</script>

<input
	{accept}
	bind:this={fileInput}
	on:change={onChange}
	style="display: none;"
	type="file"
/>
<div
	class="relative border-2 border-dashed rounded mb-2 px-3 py-7 text-center cursor-pointer {isDragging
		? 'border-green-300 bg-green-50 text-green-500'
		: 'text-gray-500'}"
	on:click={() => {
		fileInput.click();
	}}
	on:dragenter={() => {
		isDragging = true;
	}}
	on:dragleave={() => {
		isDragging = false;
	}}
	on:dragover|preventDefault
	on:drop|preventDefault={(e) => {
		isDragging = false;
		fileInput.files = e.dataTransfer?.files ?? null;
		onChange();
	}}
>
	{#if !imgSrc}
		<span class="pointer-events-none text-sm">{label}</span>
	{:else}
		<img
			alt=""
			class="pointer-events-none shadow mx-auto max-h-44"
			src={imgSrc}
		/>
	{/if}
	{#if isLoading}
		<div
			class="absolute flex items-center justify-center top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 h-12 w-12 bg-white border border-gray-100 rounded-full shadow"
		>
			<IconSpin classNames="text-purple-500 animate-spin h-6 w-6" />
		</div>
	{/if}
</div>
