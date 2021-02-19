<template>
	<div class="intro-page">
		<div class="about">
			<h1 class="about-heading">Minimax</h1>
			<div class="description">
				"Minimax (sometimes MinMax) is a decision rule used in artificial intelligence, decision
				theory, game theory, statistics, and philosophy for minimizing the possible loss for a
				worst case (maximum loss) scenario"
			</div>

			<n-link to="/tic-tac-toe" class="next-link">Continue</n-link>
		</div>
	</div>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
.intro-page {
	position: fixed;
	height: 100%;
	top: 0;
	left: 0;
	width: 100%;
	display: flex;
	justify-content: center;
	align-items: center;
	flex-direction: column;
}

.about {
	padding: 24px;
	max-width: 660px;

	.about-heading {
		@extend %h1;
	}

	.description {
		font-size: rem(18px);
	}
}

.next-link {
	@extend %link;

	color: rgb(162, 147, 248);
	margin-top: 40px;
	transition: opacity 240ms ease-out;

	&:hover {
		opacity: 0.6;
	}
}
</style>
