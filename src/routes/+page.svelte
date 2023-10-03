<script>
	import { tick } from 'svelte';
	import Individual from '$lib/Individual.svelte';
	const target = 'HELLO';
	const MAX_GENERATIONS = 1000;

	let populationSize = 100;
	let running = false;
	let generation = 0;

	function initializePopulation() {
		const pop = [];
		for (let i = 0; i < populationSize; i++) {
			pop.push(randomString());
		}
		return pop;
	}

	async function runGA() {
		while (running && generation < MAX_GENERATIONS) {
			// Create a new generation
			const newPopulation = [];
			for (let i = 0; i < populationSize; i++) {
				const parent1 = selectParent();
				const parent2 = selectParent();
				let offspring = crossover(parent1, parent2);
				// ~10% mutation chance in the offspring
				if (Math.random() < 0.1) {
					offspring = mutate(offspring);
				}
				newPopulation.push(offspring);
			}
			population = newPopulation;
			generation++;
			if (bestFitness >= target.length) {
				// perfect score
				running = false;
			}

			await new Promise(
				(resolve) =>
					setTimeout(() => {
						tick(); // To update the DOM
						resolve();
					}, 1000) // 1-second delay
			);
		}
	}

	function start() {
		running = true;
		runGA();
	}

	function stop() {
		running = false;
	}

	function reset() {
		running = false; // Stop the GA if it's running
		population = initializePopulation(); // Reset the population
		generation = 0; // Reset generation count
		bestFitness = 0; // Reset best fitness (if you're tracking this)
	}

	// Generate a random string of the same length as the target
	function randomString() {
		let result = '';
		const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
		for (let i = 0; i < target.length; i++) {
			result += characters.charAt(Math.floor(Math.random() * characters.length));
		}
		return result;
	}

	// Calculate the score for a random string based on correctness of character placement
	function fitness(individual) {
		let score = 0;
		// Determines whether the character at any location is correctly placed in the string (as compared to target)
		for (let i = 0; i < target.length; i++) {
			if (individual[i] === target[i]) score++;
		}
		return score;
	}

	// Determine whether new random is better than existing string
	function selectParent() {
		// Tournament selection for simplicity
		const tournamentSize = 5;
		// Create a new random string of 5 characters
		let best = randomString();
		for (let i = 1; i < tournamentSize; i++) {
			// Select random string from current population
			const contender = population[Math.floor(Math.random() * population.length)];
			if (fitness(contender) > fitness(best)) {
				best = contender;
			}
		}
		return best;
	}

	// Merge parts of both passed strings based on a randomly calculated midpoint
	function crossover(parent1, parent2) {
		// Midpoint is calculated randomly to determine how mcuh of parent1 and parent2 will be in the child
		const midpoint = Math.floor(Math.random() * target.length);
		return parent1.substring(0, midpoint) + parent2.substring(midpoint);
	}

	// Replace a character at a random index in the child string with another randomly generated character
	function mutate(individual) {
		const index = Math.floor(Math.random() * target.length);
		const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
		const newChar = characters.charAt(Math.floor(Math.random() * characters.length));
		return individual.substring(0, index) + newChar + individual.substring(index + 1);
	}

	// Initial population
	let population = initializePopulation();
	let bestFitness = 0;

	$: bestFitness = Math.max(...population.map(fitness));
	$: progress = (bestFitness / target.length) * 100;
</script>

<button on:click={start}>Start</button>
<button on:click={stop}>Stop</button>
<button on:click={reset}>Reset</button>

<p>Generation: {generation}</p>

<div class="progress-bar">
	<div class="filler" style="width: {progress}%" />
</div>

{#each population as individual, index (index)}
	<Individual {individual} {target} />
{/each}

<style>
	.progress-bar {
		background: grey;
		height: 20px;
	}
	.filler {
		background: green;
		height: 100%;
		transition: width 0.5s;
	}
</style>
