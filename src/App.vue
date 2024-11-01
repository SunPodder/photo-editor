<script lang="ts" setup>
import { reactive, ref, watchEffect } from "vue";

const defaultFilters = [
	{ name: "grayscale", value: 0, unit: "%", min: 0, max: 100 },
	{ name: "blur", value: 0, unit: "px", min: 0, max: 10 },
	{ name: "brightness", value: 100, unit: "%", min: 0, max: 200 },
	{ name: "contrast", value: 100, unit: "%", min: 0, max: 200 },
	{ name: "hue-rotate", value: 0, unit: "deg", min: 0, max: 360 },
	{ name: "invert", value: 0, unit: "%", min: 0, max: 100 },
	{ name: "saturate", value: 100, unit: "%", min: 0, max: 200 },
	{ name: "sepia", value: 0, unit: "%", min: 0, max: 100 },
];
const filters = reactive([
	{ name: "grayscale", value: 0, unit: "%", min: 0, max: 100 },
	{ name: "blur", value: 0, unit: "px", min: 0, max: 10 },
	{ name: "brightness", value: 100, unit: "%", min: 0, max: 200 },
	{ name: "contrast", value: 100, unit: "%", min: 0, max: 200 },
	{ name: "hue-rotate", value: 0, unit: "deg", min: 0, max: 360 },
	{ name: "invert", value: 0, unit: "%", min: 0, max: 100 },
	{ name: "saturate", value: 100, unit: "%", min: 0, max: 200 },
	{ name: "sepia", value: 0, unit: "%", min: 0, max: 100 },
]);

const selectedImage = ref(null);
const selectedFilter = ref(0);
const filter = ref("");

watchEffect(() => {
	filter.value = "";
	for (const obj of filters) {
		filter.value += `${obj.name}(${obj.value}${obj.unit}) `;
	}
	filter.value = filter.value.trim();
});

const reset = () => {
	for (const i in filters) {
		filters[i].value = defaultFilters[i].value;
	}
}

const handleImageDrop = (event) => {
	event.preventDefault();
	const file = event.dataTransfer.files[0];
	loadImage(file);
};

const handleImageSelect = (event) => {
	const file = event.target.files[0];
	loadImage(file);
};

const loadImage = (file) => {
	if (file && file.type.startsWith("image/")) {
		const reader = new FileReader();
		reader.onload = (e) => {
			selectedImage.value = e.target.result;
		};
		reader.readAsDataURL(file);
	}
	reset();
};

const downloadImage = () => {
	if (!selectedImage.value) return;
	const canvas = document.createElement("canvas");
	const img = new Image();
	img.src = selectedImage.value;
	img.onload = () => {
		canvas.width = img.width;
		canvas.height = img.height;
		const ctx = canvas.getContext("2d");
		if (!ctx) return;
		ctx.filter = filter.value;
		ctx.drawImage(img, 0, 0);
		const link = document.createElement("a");
		let name = selectedImage.value.split("/").pop();
		name = name.split("?")[0];
		name = name.split(".")[0];
		link.download = `${name}-filtered.png`;
		link.href = canvas.toDataURL("image/png");
		link.click();
	};
};

</script>

<template>
	<main>
		<div id="image-view">
			<div
				id="drop-area"
				@dragover.prevent
				@drop="handleImageDrop"
				@click="() => $refs.fileInput.click()"
				:style="{
					border: selectedImage ? 'none' : '2px dashed #ccc',
					placeItems: !selectedImage ? 'center' : 'initial',
				}"
			>
				<p v-if="!selectedImage">
					Drag and drop an image here, or click to select one
				</p>
				<img
					v-if="selectedImage"
					:src="selectedImage"
					:style="{
						filter,
						maxWidth: '100%',
						maxHeight: '100%',
						objectFit: 'contain',
					}"
					alt="Selected Image"
				/>
				<input
					type="file"
					ref="fileInput"
					@change="handleImageSelect"
					style="display: none"
				/>
			</div>
		</div>
		<div id="aside">
			<h2>Filters</h2>
			<ul>
				<li
					v-for="(filter, i) in filters"
					:key="i"
					@click="selectedFilter = i"
					:style="{
						backgroundColor: selectedFilter === i ? 'hsl(0, 0%, 40%)' : 'transparent',
						color: selectedFilter === i ? 'white' : 'hsl(0, 0%, 40%)',
					}"
				>
					{{
						filter.name.charAt(0).toUpperCase() +
						filter.name.slice(1)
					}}
				</li>
			</ul>
			<div>
				<input
					type="range"
					:min="filters[selectedFilter].min"
					:max="filters[selectedFilter].max"
					:value="filters[selectedFilter].value"
					@input="filters[selectedFilter].value = +(($event.target as HTMLInputElement).value)"
				/>
			</div>
			<div>
				<button @click="reset">
					Reset
				</button>
				<button @click="downloadImage">
					Save
				</button>
			</div>
		</div>
	</main>
</template>

<style scoped>
main {
	height: 100vh;
	width: 100%;
	display: grid;

	grid-template-columns: 1fr 0.5fr;
}

#image-view {
	min-width: 100%;
	min-height: 100%;
	display: grid;
	place-items: center;
}

#drop-area {
	text-align: center;
	cursor: pointer;
	width: 60%;
	aspect-ratio: 1/1;
	display: grid;
	place-items: center;
}

#aside {
	text-align: center;
	padding: 6rem 1rem;
	display: flex;
	flex-direction: column;
}

ul {
	list-style-type: none;
	display: flex;
	flex-wrap: wrap;
	gap: 0.5rem;
}

li {
	padding: 1rem;
	border: 1px solid hsl(0, 0%, 40%);
	border-radius: 0.5rem;
	cursor: pointer;
}

input {
	width: 80%;
	accent-color: hsl(0, 0%, 40%);
}

button {
	font-size: 1rem;
	margin: 1rem;
	padding: 0.5rem 1rem;
	border: none;
	border-radius: 0.5rem;
	background-color: hsl(0, 0%, 40%);
	color: white;
	cursor: pointer;
}

</style>
