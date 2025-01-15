<script>
	import Timer from './Timer.svelte';
	import Actions from './Actions.svelte';
	import { hintName } from "@sudoku/stores/hints";
</script>

<!-- <div class="action-bar space-y-3 xs:space-y-0">
	<Timer />
	<HintName />

	<Actions />
</div> -->

<div class="action-bar space-y-3">
	<div class="flex justify-between">
		<div></div> <!-- 占位元素，用于将文本推到右侧 -->
		<span class="text-2xl text-gray-500" class:opacity-0={$hintName === 'NULL STR'} class:opacity-50={$hintName !== 'NULL STR'}>
			{$hintName}
		</span>
	</div>

	<div class="flex justify-between">
		<Timer />
		<Actions />
	</div>
</div>

<style>
	.action-bar {
		@apply flex flex-col flex-wrap justify-between pb-4;
	}

	/* @screen xs {
		.action-bar {
			@apply flex-row;
		}
	} */
</style>