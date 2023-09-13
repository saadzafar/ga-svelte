<script>
	export let individual;
	export let target;
</script>

<p>
	{#each individual.split('') as char, index}
		<span class={target[index] === char ? 'correct' : 'incorrect'}>{char}</span>
	{/each}
</p>

<style>
	.correct {
		color: green;
	}
	.incorrect {
		color: red;
	}
</style>
