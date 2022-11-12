<script lang="ts">
	let intervalId: any = undefined;
	type Position = [number, number];
	export let BOARD_WIDTH = 10;
	export let BOARD_HEIGHT = 10;
	$: score = game.snake.body.length - 1;
	let TOTOL_APPLE_AMOUNT = 4;

	type GameState = {
		isOver: boolean;
		snake: {
			body: Position[];
			direction: [0, -1] | [0, 1] | [-1, 0] | [1, 0];
		};
		apple: {
			locations: Position[];
		};
		interval: number;
	};

	export let game = initGame();
	function genPositions(positions: Position[]): Position[] {
		if (positions.length >= TOTOL_APPLE_AMOUNT) return positions;
		const seed = Math.floor(Math.random() * BOARD_HEIGHT * BOARD_WIDTH);
		if (positions.map(([x, y]) => x * BOARD_WIDTH + y).includes(seed)) {
			return genPositions(positions);
		} else {
			return genPositions([...positions, [seed % BOARD_WIDTH, Math.floor(seed / BOARD_WIDTH)]]);
		}
	}

	function initGame(): GameState {
		return {
			isOver: false,
			snake: {
				body: [[0, 0]],
				direction: [1, 0]
			},
			apple: { locations: genPositions([]) },
			interval: 500
		};
	}

	type BoardCell = {
		hasFruit: boolean;
		hasSnakeBody: boolean;
		hasSnakeHead: boolean;
	};
	$: board = Array(BOARD_WIDTH)
		.fill(undefined)
		.map((col, x) =>
			Array(BOARD_HEIGHT)
				.fill(undefined)
				.map((cell, y) => ({
					hasFruit: game.apple.locations.some((l) => JSON.stringify(l) === JSON.stringify([x, y])),
					hasSnakeBody: game.snake.body.some(
						(b, i) => !!i && JSON.stringify(b) === JSON.stringify([x, y])
					),
					hasSnakeHead: game.snake.body.some(
						(b, i) => i === 0 && JSON.stringify(b) === JSON.stringify([x, y])
					)
				}))
		);
	function isSnakeIstTot(body: Position[]) {
		return (
			body[0][0] < 0 ||
			body[0][0] >= BOARD_WIDTH ||
			body[0][1] < 0 ||
			body[0][1] >= BOARD_HEIGHT ||
			body.slice(1).some(([x, y]) => body[0][0] === x && body[0][1] === y)
		);
	}

	function updateGame() {
		const newAppleLocations = game.apple.locations.filter(([x, y]) => !board[x][y].hasSnakeHead);
		const snakeHead = game.snake.body[0];
		const hasSnakeEaten = game.apple.locations.length > newAppleLocations.length;
		const newSnakeBody = [
			[snakeHead[0] + game.snake.direction[0], snakeHead[1] + game.snake.direction[1]] as Position
		].concat(hasSnakeEaten ? game.snake.body : game.snake.body.slice(0, -1));
		console.log({ newSnakeBody, hasSnakeEaten, snakeHead });

		game.apple.locations = genPositions(newAppleLocations);

		if (hasSnakeEaten) {
			game.interval = game.interval * 0.95;
		}

		if (isSnakeIstTot(newSnakeBody)) {
			game.isOver = true;
		} else {
			game.snake.body = newSnakeBody;
		}
	}

	function onKeyDown(e: KeyboardEvent) {
		switch (e.keyCode) {
			case 38:
				game.snake.direction = [0, -1];
				break;
			case 40:
				game.snake.direction = [0, 1];
				break;
			case 37:
				game.snake.direction = [-1, 0];
				break;
			case 39:
				game.snake.direction = [1, 0];
				break;
		}
	}

	function tickGame() {
		if (!game.isOver) {
			updateGame();
			setTimeout(() => {
				tickGame();
			}, game.interval);
		}
	}

	function restartGame() {
		game = initGame();
	}

	function getCellText(cell: BoardCell) {
		if (cell.hasSnakeHead) return game.isOver ? '💀' : '🐸';
		if (cell.hasSnakeBody) return game.isOver ? '🟥' : '🟩';
		if (cell.hasFruit) return '🍎';
		return '⬜️';
	}
</script>

<form class="flex justify-content align-center  flex-col w-64">
	<h1>Snake Game</h1>
	{#if game.isOver}
		<h2>Game Over</h2>
		<button on:click|preventDefault={restartGame}>Restart</button>
	{:else}
		<button on:click|preventDefault={tickGame}>Start</button>
	{/if}
	<h2>Score: {score}</h2>

	<div class="grid">
		{#each board as col, y (y)}
			{#each col as cell, x (x)}
				<div class="gridcell">{getCellText(board[x][y])}</div>
			{/each}
		{/each}
	</div>
</form>
<svelte:window on:keydown|preventDefault={onKeyDown} />

<style>
	.grid {
		display: grid;
		grid-template-columns: repeat(10, 1fr);
	}
	.gridcell {
		display: flex;
		width: 2rem;
	}
</style>
