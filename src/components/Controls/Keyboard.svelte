<script>
	import { userGrid } from '@sudoku/stores/grid';
	import { hints,hintCells,hintName } from '@sudoku/stores/hints';
	import { cursor } from '@sudoku/stores/cursor';
	import { notes } from '@sudoku/stores/notes';
	import { candidates } from '@sudoku/stores/candidates';
	import {undoRedoManager} from '@sudoku/stores/unredomanager';

	// TODO: Improve keyboardDisabled
	import { keyboardDisabled, candidatesNumDisable } from '@sudoku/stores/keyboard';

	function handleKeyButton(num) {
		console.log('handleKeyButton', num)
		if (!$keyboardDisabled) {
			if ($notes) {
				if (num === 0) {
					candidates.clear($cursor);
				} else {
					candidates.add($cursor, num);
				}
				userGrid.set($cursor, 0);
			} else {
				// 重点改这里

				// if ($candidates.hasOwnProperty($cursor.x + ',' + $cursor.y)) {
				// 	candidates.clear($cursor);
				// }

				userGrid.set($cursor, num);
				// 用户从下方键盘输入而并非双击Cell时，也应该更新hintCells
				const key = $cursor.y + ',' + $cursor.x
				if ($hintCells.hasOwnProperty(key)){
					hintCells.updateOneCells(key, num) // 先删掉该候选值
					hintCells.updateAllHintCells($userGrid, $hints, $hintName);
					hintCells.delete($cursor.x, $cursor.y);
					hintCells.checkAndUpdate($userGrid, $hints);
					// 粗暴做法，一旦输入，我就清空原本的候选值结果，重新求解
					console.log('用户从下方键盘输入, update grid', $hintCells)
					
				}
			}
		}
		undoRedoManager.isUndoEmptyFunc();
		undoRedoManager.isRedoEmptyFunc();
		undoRedoManager.isLastHintStateEmptyFunc();

	}

	function handleKey(e) {
		switch (e.key || e.keyCode) {
			case 'ArrowUp':
			case 38:
			case 'w':
			case 87:
				cursor.move(0, -1);
				break;

			case 'ArrowDown':
			case 40:
			case 's':
			case 83:
				cursor.move(0, 1);
				break;

			case 'ArrowLeft':
			case 37:
			case 'a':
			case 65:
				cursor.move(-1);
				break;

			case 'ArrowRight':
			case 39:
			case 'd':
			case 68:
				cursor.move(1);
				break;

			case 'Backspace':
			case 8:
			case 'Delete':
			case 46:
				handleKeyButton(0);
				break;

			default:
				if (e.key && e.key * 1 >= 0 && e.key * 1 < 10) {
					handleKeyButton(e.key * 1);
				} else if (e.keyCode - 48 >= 0 && e.keyCode - 48 < 10) {
					handleKeyButton(e.keyCode - 48);
				}
				break;
		}
	}
</script>

<svelte:window on:keydown={handleKey} /><!--on:beforeunload|preventDefault={e => e.returnValue = ''} />-->

<div class="keyboard-grid">

	{#each Array(10) as _, keyNum}
		{#if keyNum === 9}
			<button class="btn btn-key" disabled={$keyboardDisabled} title="Erase Field" on:click={() => handleKeyButton(0)}>
				<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
				</svg>
			</button>
		{:else}
			<button class="btn btn-key" disabled={$keyboardDisabled || $candidatesNumDisable.isCandateNum(keyNum + 1)} title="Insert {keyNum + 1}" on:click={() => handleKeyButton(keyNum + 1)}>
				{keyNum + 1}
			</button>
		{/if}
	{/each}

</div>


<style>
	.keyboard-grid {
		@apply grid grid-rows-2 grid-cols-5 gap-3;
	}


	.btn-key {
		@apply py-4 px-0;
	}
</style>