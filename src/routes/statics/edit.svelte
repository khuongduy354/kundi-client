<script>
	export let mode;
	export let set_name = '';
	export let flashcards = [{ word: 'word', definition: 'Definition 1' }];
	export let set_id = '';
	import { parseImport, parseExport } from '../../helpers/parse';
	import { UserState, APIUrl } from '../../store';

	let importValue = '';
	let temp_flcards = [...flashcards];
	let user = null;
	let deckTitle = '';
	let deckDescription = '';
	let terms = [];
	let definitions = [];

	let term = '';
	let defi = '';

	UserState.subscribe((value) => (user = value));

	function exportCards() {
		let result = parseExport(temp_flcards, ' ', '\n');
		let blob = new Blob([result], { type: 'text/plain' });
		const link = document.createElement('a');
		link.href = URL.createObjectURL(blob);
		link.download = 'export.txt';
		link.click();
	}
	function importCards() {
		temp_flcards = [...temp_flcards, ...parseImport(importValue, ' ', '\n')];
	}
	$: addFlashcard = (new_card) => { 
		temp_flcards.push(new_card);
		temp_flcards = temp_flcards;
	};

	// Function to handle form submission
	async function handleSubmit() {
		if (user == null) {
      alert("please login")
			return;
		}
    if (set_id==''){
      alert("please select a deck")
      return;
    }
		const queryParams = {
			token: user.idToken
		};
		const queryString = new URLSearchParams(queryParams).toString();
		let url = APIUrl + '/v1/sets/' + set_id + '/cards' + '?' + queryString;
		let body = JSON.stringify({cardList: temp_flcards});
		console.log(body);
		let res = await fetch(url, {
			method: 'POST',
			body,
			headers: {
				'Content-Type': 'application/json'
			}
		});
		if (res.ok) {
			alert('cards created');
			flashcards = temp_flcards;
		} else {
			alert('Cant make request');
		}
	}
</script>

<main>
	<header>
		<h1>Edit Deck {set_name}</h1>
		<!-- <div class="deck-info">
			<input type="text" bind:value={deckTitle} placeholder="Deck Title" class="input-field" />
			<textarea
				bind:value={deckDescription}
				placeholder="Deck Description"
				class="textarea-field"
			/>
		</div> -->
	</header>

	<!--   <div class="card-columns">
    <div class="term-column">
      <h2>Term</h2>
      <ul>
        {#each terms as term}
          <li>{term}</li>
        {/each}
      </ul>
    </div>
    <div class="definition-column">
      <h2>Definition</h2>
      <ul>
        {#each definitions as definition}
          <li>{definition}</li>
        {/each}
      </ul>
    </div>
  </div> -->
	<p>Import areas</p>
	<textarea class="import" bind:value={importValue} />
	<div class="button-container">
		<button on:click={importCards} class="button">Import</button>
		<span />
		<button on:click={exportCards} class="button">Export</button>
	</div>
	{#each temp_flcards as flashcard, idx}
		{#if flashcard.definition != '' && flashcard.word != ''}
			<div class="form-row">
				<input type="text" placeholder="Term" class="input-field" value={flashcard.word} />
				<input
					type="text"
					placeholder="Definition"
					class="input-field"
					value={flashcard.definition}
				/>
				<button
					on:click={() => {
						temp_flcards.splice(idx, 1);
						temp_flcards = temp_flcards;
					}}
					class="button"
					type="button">Delete Row</button
				>
			</div>
		{/if}
	{/each}
	<form>
		<div class="form-row">
			<input bind:value={term} type="text" placeholder="Term" class="input-field" />
			<input bind:value={defi} type="text" placeholder="Definition" class="input-field" />
			<button
				on:click={() => {
					if (term != '' || defi != '') {
						let new_word = { word: term, definition: defi };
						addFlashcard(new_word);
						term = '';
						defi = '';
					}
				}}
				class="button"
				type="button">Add Row</button
			>
			<button
				on:click={() => {
					term = '';
					defi = '';
				}}
				class="button"
				type="button">Delete Row</button
			>
		</div>
		<button on:click={handleSubmit} class="button">Submit</button>
	</form>
	<button
		on:click={() => {
			mode = 'Slideshow';
		}}
		class="button">Back</button
	>
</main>

<style>
	.import {
		width: 100%;
		height: 200px;
		padding: 8px;
		font-size: 14px;
		border: 1px solid #ccc;
		border-radius: 4px;
	}
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background-color: #27374d;
		color: #ffffff;
		padding: 20px;
		height: 100vh;
		width: 100%;
	}

	header {
		text-align: center;
		margin-bottom: 20px;
	}

	.deck-info {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-bottom: 20px;
	}

	.input-field,
	.textarea-field {
		width: 100%;
		padding: 10px;
		margin-bottom: 10px;
		border-radius: 4px;
		border: none;
	}

	.textarea-field {
		height: 120px;
	}

	.card-columns {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
		grid-gap: 20px;
		margin-bottom: 20px;
	}

	.term-column,
	.definition-column {
		width: 100%;
	}

	.button-container {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
		margin-bottom: 20px;
	}

	.button {
		padding: 10px 20px;
		border-radius: 4px;
		border: none;
		background-color: #526d82;
		color: #ffffff;
		cursor: pointer;
	}

	.form-row {
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 10px;
	}

	.form-row input[type='text'] {
		margin-right: 10px;
	}

	ul {
		list-style-type: none;
		padding: 0;
		margin: 0;
	}

	@media (max-width: 768px) {
		.card-columns {
			grid-template-columns: 1fr;
		}
	}
</style>
