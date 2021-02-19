<template>
	<div class="tic-tac-toe-page">
		<div class="game-wrapper">
			<tic-tac-toe-board :show-controls="true" />
		</div>
	</div>
</template>

<script>
import TicTacToeBoard from '~/components/TicTacToeBoard.vue'

export default {
	head: {
		title: 'Minimax'
	},
	components: {
		TicTacToeBoard
	}
}
</script>

<style lang="scss" scoped>
.game-wrapper {
	padding: 64px 24px;
	min-height: 100vh;
	display: flex;
	align-items: center;
}
</style>
