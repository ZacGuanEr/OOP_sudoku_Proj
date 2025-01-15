<script>
	import { userGrid } from "@sudoku/stores/grid";
	import { cursor } from "@sudoku/stores/cursor";
	import { hints, hintCells } from "@sudoku/stores/hints";
	import { settings } from "@sudoku/stores/settings";
	import { hintDisabled } from "@sudoku/stores/keyboard";
	import { gamePaused } from "@sudoku/stores/game";
	import {
		isRedoEmpty,
		isUndoEmpty,
		isLastHintStateEmpty,
		undoRedoManager,
	} from "@sudoku/stores/unredomanager";

	$: hintsAvailable = $hints > -1;

	function handleHint() {
		// console.log('in Action.svelte, candidates: ', $candidates);
		hints.useHint();
		if ($hints !== 0) {
			//deep copy userGrid
			let newGrid = [];
			for (let i = 0; i < 9; i++) {
				newGrid[i] = [];
				for (let j = 0; j < 9; j++) {
					newGrid[i][j] = $userGrid[i][j];
				}
			}
			undoRedoManager.pushLastHintState({
                        grid: newGrid,
                    });
			userGrid.applyHint($cursor, $hints);
		}
		undoRedoManager.isLastHintStateEmptyFunc();
		undoRedoManager.isRedoEmptyFunc();
		undoRedoManager.isUndoEmptyFunc();
	}

	function ActionUndo() {
		hintCells.clear();
		userGrid.Undo();
	}

	function ActionRedo() {
		hintCells.clear();
		userGrid.Redo();
	}

	function returnToLastHintState() {
		hintCells.clear();
		hints.reset();
		userGrid.returnHintAll();
		undoRedoManager.clearUndo();
		undoRedoManager.isUndoEmptyFunc();
		hints.useHint();
		if ($hints !== 0) {
			userGrid.applyHint($cursor, $hints);
		}
	}
</script>

<div class="action-buttons space-x-3">
	<button
		class="btn btn-round btn-badge"
		on:click={() => returnToLastHintState()}
		title="Return to Last Hint State"
		disabled={$gamePaused || $isLastHintStateEmpty || $hints === 0}
	>
		<svg
			class="icon-outline"
			xmlns="http://www.w3.org/2000/svg"
			fill="none"
			viewBox="0 0 24 24"
			stroke="currentColor"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M2.5 2v6h6M2.66 15.57a10 10 0 1 0 .57-8.38"
			/>
		</svg>
	</button>

	<button
		class="btn btn-round"
		disabled={$gamePaused || $isUndoEmpty}
		title="Undo"
		on:click={() => ActionUndo()}
	>
		<svg
			class="icon-outline"
			xmlns="http://www.w3.org/2000/svg"
			fill="none"
			viewBox="0 0 24 24"
			stroke="currentColor"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M3 10h10a8 8 0 018 8v2M3 10l6 6m-6-6l6-6"
			/>
		</svg>
	</button>

	<button
		class="btn btn-round"
		disabled={$gamePaused || $isRedoEmpty}
		title="Redo"
		on:click={() => ActionRedo()}
	>
		<svg
			class="icon-outline"
			xmlns="http://www.w3.org/2000/svg"
			fill="none"
			viewBox="0 0 24 24"
			stroke="currentColor"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M21 10h-10a8 8 90 00-8 8v2M21 10l-6 6m6-6l-6-6"
			/>
		</svg>
	</button>

	<button
		class="btn btn-round btn-badge"
		disabled={$hintDisabled || !hintsAvailable}
		on:click={handleHint}
		title="Hints ({$hints})"
	>
		<svg
			class="icon-outline"
			xmlns="http://www.w3.org/2000/svg"
			fill="none"
			viewBox="0 0 24 24"
			stroke="currentColor"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"
			/>
		</svg>

		{#if $settings.hintsLimited}
			<span class="badge" class:badge-primary={hintsAvailable}
				>{$hints}</span
			>
		{/if}
	</button>
</div>

<!-- <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#000000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2.5 2v6h6M2.66 15.57a10 10 0 1 0 .57-8.38"/></svg> -->

<style>
	.action-buttons {
		@apply flex flex-wrap justify-evenly self-end;
	}

	.btn-badge {
		@apply relative;
	}

	.badge {
		min-height: 20px;
		min-width: 20px;
		@apply p-1 rounded-full leading-none text-center text-xs text-white bg-gray-600 inline-block absolute top-0 left-0;
	}

	.badge-primary {
		@apply bg-primary;
	}
</style>
