<script lang="ts">
	import { emoji } from './emoji';

	type GameState = 'start' | 'playing' | 'paused' | 'won' | 'lost';

	let state: GameState = 'start';

	let size = 20;
	let grid = createGrid();

	let maxMatches = grid.length / 2;
	let selected: number[] = [];
	let matches: string[] = [];
	let time = 60;
	let timerId: number | null = null;

	function startGameTimer() {
		function countdown() {
			state !== 'paused' && (time -= 1);
		}

		timerId = setInterval(countdown, 1000);
	}

	function createGrid() {
		let cards = new Set<string>();
		let maxSize = size / 2;

		while (cards.size < maxSize) {
			const randomIdx = Math.floor(Math.random() * emoji.length);
			cards.add(emoji[randomIdx]);
		}

		return shuffle([...cards, ...cards]);
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5);
	}

	function selectCard(cardIdx: number) {
		selected = selected.concat(cardIdx);
	}

	function matchCards() {
		const [first, second] = selected;

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first]);
		}

		setTimeout(() => {
			selected = [];
		}, 500);
	}

	function resetGame() {
		timerId && clearInterval(timerId);
		timerId = null;
		time = 60;
		matches = [];
		selected = [];
		grid = createGrid();
		maxMatches = grid.length / 2;
	}

	function gameWon() {
		state = 'won';
		resetGame();
	}

	function gameLost() {
		state = 'lost';
		resetGame();
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (state) {
				case 'playing':
					state = 'paused';
					break;
				case 'paused':
					state = 'playing';
					break;

				default:
					break;
			}
		}
	}

	$: selected.length === 2 && matchCards();
	$: maxMatches === matches.length && gameWon();
	$: if (state === 'playing') {
		!timerId && startGameTimer();
	}

	$: time === 0 && gameLost();
</script>

<svelte:window on:keydown={pauseGame} />

<div class="container">
	{#if state === 'start'}
		<h1>Matching Game</h1>
		<button on:click={() => (state = 'playing')}>Play</button>
	{/if}

	{#if state === 'playing'}
		<h1 class="timer" class:pulse={time <= 10}>{time}</h1>
		<div class="matches">
			{#each matches as match}
				<div>{match}</div>
			{/each}
		</div>
		<div class="cards">
			{#each grid as card, cardIdx}
				{@const isSelected = selected.includes(cardIdx)}
				{@const isSelectedOrMatch = selected.includes(cardIdx) || matches.includes(card)}
				{@const isMatched = matches.includes(card)}
				<button
					class="card"
					class:selected={isSelected}
					disabled={isSelectedOrMatch}
					class:flip={isSelectedOrMatch}
					on:click={() => selectCard(cardIdx)}
				>
					<div class="back" class:match={isMatched}>{card}</div>
				</button>
			{/each}
		</div>
	{/if}

	{#if state === 'lost'}
		<h1>You lost! 💩</h1>
		<button on:click={() => (state = 'playing')}>Play again</button>
	{/if}

	{#if state === 'won'}
		<h1>You win! 🎉</h1>
		<button on:click={() => (state = 'playing')}>Play again</button>
	{/if}

	{#if state === 'paused'}
		<h1>Game Paused</h1>
		<button on:click={() => (state = 'playing')}>Resume</button>
	{/if}
</div>

<style>
	.container {
		width: 100%;
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		place-content: center;
	}

	.cards {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 0.4rem;
	}

	.card {
		height: 140px;
		width: 140px;
		font-size: 4rem;
		background-color: var(--bg-2);
		transition: rotate 0.3s ease-out;
		transform-style: preserve-3d;

		&.selected {
			border: 4px solid var(--border);
		}

		& .match {
			transition: opacity 0.3s ease-out;
			opacity: 0.4;
		}

		&.flip {
			rotate: y 180deg;
			pointer-events: none;
		}

		& .back {
			position: absolute;
			inset: 0;
			display: grid;
			place-content: center;
			backface-visibility: hidden;
			rotate: y 180deg;
		}
	}

	.matches {
		display: flex;
		gap: 1rem;
		margin-block: 2rem;
		font-size: 3rem;
	}

	.timer {
		transition: color 0.3s ease;
	}

	.pulse {
		color: var(--pulse);
		animation: pulse 1s infinite ease;
	}

	@keyframes pulse {
		to {
			scale: 1.4;
		}
	}
</style>
