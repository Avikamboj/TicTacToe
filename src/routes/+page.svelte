<script lang="ts">
	import { writable } from 'svelte/store';

	const userSymbol = writable<string | null>(null);
	let computerSymbol: string | null = null;
	let userPlaysFirst = writable<boolean>(true); // New state variable

	let board: (string | null)[] = Array(9).fill(null);
	let currentPlayer: string = 'X';
	let winner: string | null = null;

	const winningCombinations: number[][] = [
		[0, 1, 2],
		[3, 4, 5],
		[6, 7, 8],
		[0, 3, 6],
		[1, 4, 7],
		[2, 5, 8],
		[0, 4, 8],
		[2, 4, 6]
	];

	let isUserTurn: boolean = true;

	// Check for winner or tie
	const checkWinner = (): boolean => {
		for (let combo of winningCombinations) {
			const [a, b, c] = combo;
			if (board[a] && board[a] === board[b] && board[a] === board[c]) {
				winner = board[a];
				return true;
			}
		}

		if (board.every((cell) => cell !== null)) {
			winner = 'tie';
			return true;
		}

		return false;
	};

	// Reset the game
	const resetGame = (): void => {
		board = Array(9).fill(null);
		currentPlayer = 'X';
		winner = null;
		isUserTurn = $userPlaysFirst; // Set turn based on user preference
	};

	// Minimax algorithm for finding the best move
	const minimax = (
		newBoard: (string | null)[],
		isMaximizing: boolean
	): { score: number; index?: number } => {
		// Check if game has a winner
		if (checkForWinner(newBoard, computerSymbol)) return { score: 10 };
		if (checkForWinner(newBoard, $userSymbol)) return { score: -10 };
		if (newBoard.every((cell) => cell !== null)) return { score: 0 };

		let bestMove: number | undefined;
		let bestScore: number = isMaximizing ? -Infinity : Infinity;

		newBoard.forEach((cell, index) => {
			if (cell === null) {
				newBoard[index] = isMaximizing ? computerSymbol : $userSymbol;
				const score = minimax(newBoard, !isMaximizing).score;
				newBoard[index] = null;

				if (isMaximizing) {
					if (score > bestScore) {
						bestScore = score;
						bestMove = index;
					}
				} else {
					if (score < bestScore) {
						bestScore = score;
						bestMove = index;
					}
				}
			}
		});

		return { score: bestScore, index: bestMove };
	};

	// Helper function to check for a winner in a hypothetical board
	const checkForWinner = (board: (string | null)[], symbol: string | null): boolean => {
		for (let combo of winningCombinations) {
			const [a, b, c] = combo;
			if (board[a] === symbol && board[b] === symbol && board[c] === symbol) {
				return true;
			}
		}
		return false;
	};

	// Computer move logic
	// Computer move logic with randomness for the first move
	const computerMove = (): number | undefined => {
		if (board.every((cell) => cell === null)) {
			// First move (choose randomly from available spots)
			const availableMoves = board
				.map((cell, index) => (cell === null ? index : null))
				.filter((index) => index !== null);
			const randomMove = availableMoves[Math.floor(Math.random() * availableMoves.length)];
			return randomMove;
		} else {
			// Subsequent moves, use minimax to decide
			return minimax([...board], true).index;
		}
	};

	// Handle cell click for user
	const handleClick = (index: number): void => {
		if (board[index] || winner || !isUserTurn) return;

		board[index] = $userSymbol;
		isUserTurn = false;

		if (checkWinner()) {
			endGame();
			return;
		}

		currentPlayer = currentPlayer === 'X' ? 'O' : 'X';

		// Computer's turn
		setTimeout(() => {
			const move = computerMove();
			if (move !== undefined) board[move] = computerSymbol;

			if (checkWinner()) {
				endGame();
				return;
			}

			currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
			isUserTurn = true;
		}, 500);
	};

	// End the game and reset after 4 seconds
	const endGame = (): void => {
		if (winner === $userSymbol) {
			winner = 'You Win';
		} else if (winner === 'tie') {
			winner = "It's a Tie";
		} else {
			winner = 'You Lose';
		}

		setTimeout(() => {
			resetGame();
			$userSymbol = null;
			computerSymbol = null;
		}, 4000);
	};

	// Start the game with selected symbol
	const startGame = (symbol: string): void => {
		$userSymbol = symbol;
		computerSymbol = symbol === 'X' ? 'O' : 'X';
		resetGame();
		if (!$userPlaysFirst) {
			// Computer moves first
			setTimeout(() => {
				const move = computerMove();
				if (move !== undefined) board[move] = computerSymbol;
				isUserTurn = true;
			}, 500);
		}
	};
</script>

<!-- Game -->
{#if !$userSymbol}
	<div class="flex min-h-screen flex-col items-center justify-center bg-gray-200">
		<h1 class="mb-6 text-4xl font-bold">Tic Tac Toe</h1>
		<div class="mb-4 flex items-center">
			<input type="checkbox" id="userPlaysFirst" class="mr-2" bind:checked={$userPlaysFirst} />
			<label for="userPlaysFirst" class="text-2xl">I want to play first</label>
		</div>
		<h2 class="mb-4 text-2xl">Choose Your Mark:</h2>
		<div class="flex">
			<button
				class="mx-2 rounded bg-blue-500 px-6 py-2 text-white hover:bg-blue-700"
				on:click={() => startGame('X')}
			>
				Play as X
			</button>
			<button
				class="mx-2 rounded bg-green-500 px-6 py-2 text-white hover:bg-green-700"
				on:click={() => startGame('O')}
			>
				Play as O
			</button>
		</div>
	</div>
{/if}

{#if $userSymbol}
	<div class="flex min-h-screen flex-col items-center justify-center bg-gray-200">
		<div class="mb-6 text-xl">
			You are <span class="font-bold text-blue-600">{$userSymbol}</span> | Computer is
			<span class="font-bold text-red-600">{computerSymbol}</span>
		</div>
		<h2 class="mb-4 text-3xl">Current Player: {isUserTurn ? 'You' : 'Computer'}</h2>
		<div class="grid grid-cols-3 gap-2">
			{#each board as cell, i}
				<!-- svelte-ignore a11y_click_events_have_key_events -->
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<div
					class="flex h-24 w-24 cursor-pointer items-center justify-center border bg-white text-4xl font-bold hover:bg-gray-100"
					class:opacity-50={!!cell || !!winner}
					on:click={() => handleClick(i)}
				>
					{cell}
				</div>
			{/each}
		</div>

		{#if winner}
			<div
				class="mt-6 text-4xl font-bold"
				class:text-green-600={winner === 'You Win'}
				class:text-blue-600={winner === "It's a Tie"}
				class:text-red-600={winner === 'You Lose'}
			>
				{winner}
			</div>
		{/if}
	</div>
{/if}
