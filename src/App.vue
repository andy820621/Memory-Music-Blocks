<script setup>
import { reactive, ref } from "vue";
</script>

<template>
	<div class="infos">
		<h1>Memory Blocks</h1>
		<h3 class="status"></h3>
	</div>

	<div class="blocks">
		<div class="block"></div>
		<div class="block"></div>
		<div class="block"></div>
		<div class="block"></div>
	</div>

	<div class="inputStatus"></div>
</template>

<style lang="scss" scoped>
$colorRed: hsl(0, 100%, 66%);
$colorYellow: #ffc429;
$colorBlue: #5980c1;
$colorWhite: #dad2b1;

.infos {
	position: absolute;
	top: 0;
	left: 1.24rem;
	h1 {
		font-size: 40px;
		letter-spacing: 1px;
		margin: 0 auto;
		line-height: 1.2;
	}
	h3 {
		font-weight: 500;
		color: $colorRed;
	}
}
.blocks {
	--grid-number: 2;
	--block-size: 8rem;
	--grid-gap: 2rem;

	display: grid;
	grid-template-columns: repeat(var(--grid-number), var(--block-size));
	grid-template-rows: repeat(var(--grid-number), var(--block-size));
	gap: var(--grid-gap);

	.block {
		--block-color: 0 100% 66%;

		width: var(--block-size);
		height: var(--block-size);
		left: calc(
			var(--grid-gap) + var(--x) * (var(--grid-gap) + var(--block-size))
		);
		top: calc(
			var(--grid-gap) + var(--y) * (var(--grid-gap) + var(--block-size))
		);

		border: 1px solid;
		border-color: hsl(var(--block-color));
		filter: brightness(80%);

		box-shadow: 0 0 24px 4px hsl(var(--block-color) / 0.14);
		animation: shadow-animation 1.2s infinite alternate-reverse;
	}

	@keyframes shadow-animation {
		100% {
			box-shadow: 0 0 24px 4px hsl(var(--block-color) / 0.44);
		}
	}
}
</style>
