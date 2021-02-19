<template>
	<div class="tic-tact-toe-game">
		<div class="interface-wrapper">
			<div class="controls" v-if="showControls">
				<button :disabled="gameStarted" @click="toggleFirstPlayer">
					First Turn: {{ humanMove == 'X' ? 'You' : 'AI' }}
				</button>
				<button @click="toggleGame()">
					{{ this.gameStarted ? 'Restart' : 'Start' }}
				</button>
			</div>

			<div class="board-wrapper">
				<button
					v-for="i in 9"
					:key="`button-${i}`"
					@click="playMove(i - 1, humanMove)"
					@mouseenter="checkOutcomes(i - 1)"
					@mouseleave="removeOutcome()"
					:disabled="state[i - 1] || turn != humanMove || !gameStarted"
				>
					<span>{{ state[i - 1] }}</span>
				</button>
			</div>

			<div class="outcome-wrapper" v-if="showControls">
				<span>Ai wins {{ aiWins }}</span
				><span>Draws {{ draws }}</span
				><span>Human wins {{ humanWins }}</span>
			</div>

			<div v-if="winner" class="winner-wrapper">
				{{ winner == 'X' || winner == 'O' ? 'Winner: ' : '' }} {{ winner }}
			</div>
		</div>

		<div class="hints-wrapper">
			<label class="toggle-hints-wrapper">
				<input type="checkbox" v-model="hints" />
				Enable Hints When Hovering Square
			</label>

			<div class="outcome-checker" v-show="hints">
				{{
					outcome ? `Playing this square ${outcome}` : 'Hover over a square to show outcomes'
				}}
			</div>
		</div>
	</div>
</template>

<script>
export default {
	props: {
		showControls: {
			type: Boolean,
			default: false
		},
		autoplay: {
			type: Boolean,
			default: true
		}
	},
	data: () => ({
		state: [null, null, null, null, null, null, null, null, null],
		humanMove: 'O',
		aiMove: 'X',
		turn: 'X',
		gameStarted: false,
		humanWins: 0,
		aiWins: 0,
		draws: 0,
		winner: null,
		outcome: '',
		hints: false
	}),
	methods: {
		toggleFirstPlayer() {
			if (this.humanMove == 'X') {
				this.humanMove = 'O'
				this.aiMove = 'X'
			} else {
				this.aiMove = 'O'
				this.humanMove = 'X'
			}
		},
		startGame() {
			this.gameStarted = true

			if (this.humanMove != 'X') {
				let moves = [0, 2, 6, 8]
				this.playMove(moves[Math.floor(Math.random() * moves.length)], 'X')
			}
		},
		toggleGame() {
			if (this.gameStarted) {
				this.restartGame()
			} else {
				this.startGame()
			}
		},
		checkOutcomes(index) {
			const previewBoard = [...this.state]
			previewBoard[index] = this.humanMove

			const outcome = this.getBestMove(previewBoard, this.aiMove).score

			if (outcome == -1 || outcome == 1) {
				this.outcome = 'will result in a loss'
			} else {
				this.outcome = 'can result in a draw'
			}
		},
		removeOutcome() {
			this.outcome = ''
		},
		playMove(index, player) {
			this.state[index] = player

			const winner = this.calcWinner(this.state)

			if (winner) {
				this.updateWins(winner)

				this.winner = winner
			} else {
				this.turn = this.turn == 'X' ? 'O' : 'X'

				if (this.turn != this.humanMove) {
					this.playMove(this.getBestMove(this.state).move, this.aiMove)
				}
			}
		},
		updateWins(winner) {
			if (this.humanMove == winner) {
				this.humanWins++
			} else if (this.aiMove == winner) {
				this.aiWins++
			} else {
				this.draws++
			}
		},
		calcWinner(board) {
			const s = board

			if (s[0] && s[0] == s[1] && s[0] == s[2]) {
				return s[0]
			} else if (s[3] && s[3] == s[4] && s[3] == s[5]) {
				return s[3]
			} else if (s[6] && s[6] == s[7] && s[6] == s[8]) {
				return s[6]
			} else if (s[0] && s[0] == s[3] && s[0] == s[6]) {
				return s[0]
			} else if (s[1] && s[1] == s[4] && s[1] == s[7]) {
				return s[1]
			} else if (s[2] && s[2] == s[5] && s[2] == s[8]) {
				return s[2]
			} else if (s[0] && s[0] == s[4] && s[0] == s[8]) {
				return s[0]
			} else if (s[2] && s[2] == s[4] && s[2] == s[6]) {
				return s[2]
			}

			// Draw
			let isDraw = 'draw'
			board.forEach(val => {
				if (!val) isDraw = false
			})

			return isDraw
		},
		restartGame() {
			this.gameStarted = false
			this.state = [null, null, null, null, null, null, null, null, null]
			this.turn = 'X'
			this.winner = null
			this.outcome = ''
		},
		getBestMove(board) {
			const availSpots = this.emptySquares(board)

			if (this.aiMove == 'X') {
				let bestScore = -Infinity
				let bestMove = null
				for (let i = 0; i < availSpots.length; i++) {
					const newBoard = [...board]
					newBoard[availSpots[i]] = 'X'
					const score = this.minimax(newBoard, 0, false)

					if (score > bestScore) {
						bestScore = score
						bestMove = availSpots[i]
					}
				}

				return { move: bestMove, score: bestScore }
			} else {
				let bestScore = Infinity
				let bestMove = null
				for (let i = 0; i < availSpots.length; i++) {
					const newBoard = [...board]
					newBoard[availSpots[i]] = 'O'
					const score = this.minimax(newBoard, 0, true)

					if (score < bestScore) {
						bestScore = score
						bestMove = availSpots[i]
					}
				}

				return { move: bestMove, score: bestScore }
			}
		},
		emptySquares(board) {
			let availSpots = []
			board.forEach((pos, i) => {
				if (!pos) availSpots.push(i)
			})

			return availSpots
		},
		minimax(board, depth, isMaximizer) {
			const winner = this.calcWinner(board)

			if (winner) {
				if (winner == 'draw') {
					return 0
				}

				return winner == 'X' ? 1 : -1
			}

			const availSpots = this.emptySquares(board)

			if (isMaximizer) {
				let bestScore = -Infinity

				for (let i = 0; i < availSpots.length; i++) {
					const newBoard = [...board]
					newBoard[availSpots[i]] = 'X'

					bestScore = Math.max(bestScore, this.minimax(newBoard, depth + 1, false))
				}

				return bestScore
			} else {
				let bestScore = Infinity

				for (let i = 0; i < availSpots.length; i++) {
					const newBoard = [...board]
					newBoard[availSpots[i]] = 'O'

					bestScore = Math.min(bestScore, this.minimax(newBoard, depth + 1, true))
				}

				return bestScore
			}
		}
	}
}
</script>

<style lang="scss" scoped>
.tic-tact-toe-game {
	align-items: center;
	display: flex;
	justify-content: center;
	left: 0;
	top: 0;
	width: 100%;
}

.interface-wrapper {
	max-width: 660px;
	width: 60%;
}

.hints-wrapper {
	padding-left: 40px;
	width: 40%;
	max-width: 330px;

	label {
		align-items: center;
		display: flex;
		font-size: rem(14px);
	}

	input {
		height: 14px;
		width: 14px;
	}

	.outcome-checker {
		font-size: rem(24px);
		margin-top: 24px;
	}
}

.controls {
	display: flex;
	justify-content: space-between;
	margin-bottom: 40px;

	button[disabled] {
		opacity: 0.3;
	}
}

.outcome-wrapper {
	display: flex;
	justify-content: space-between;
	margin-top: 40px;
}

.board-wrapper {
	display: flex;
	flex-wrap: wrap;
	width: 100%;

	button {
		cursor: pointer;
		box-sizing: border-box;
		display: block;
		position: relative;
		width: calc(33.333%);
		border: 2px solid rgba(rgb(191, 181, 247), 0.3);
		background-color: #fcfcfc;
		transition: background-color 240ms ease-out;

		&[disabled] {
			cursor: initial;
			background-color: rgba(rgb(191, 181, 247), 0.15);
		}

		&::before {
			content: '';
			display: block;
			padding-top: 100%;
			width: 100%;
		}

		&:not([disabled]):hover {
			background-color: #ecedf7;
		}

		span {
			align-items: center;
			justify-content: center;
			font-size: rem(100px);
			font-weight: 700;
			display: flex;
			height: 100%;
			width: 100%;
			left: 0;
			top: 0;
			position: absolute;
			color: rgb(137, 118, 245);
		}
	}
}

.winner-wrapper {
	text-align: center;
	font-size: rem(32px);
	font-weight: 700;
	margin-top: 40px;
}
</style>
