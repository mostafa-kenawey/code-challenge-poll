<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import { get } from 'svelte/store';

	const API_BASE_URL = 'http://localhost:8000';
	const { question_id } = $page.params;
	
	let answers: Answer[] = [];
	let loading = true;
	let error = '';
	let text = '';
	let message = '';

	async function handleSubmit(event: SubmitEvent) {
		event.preventDefault();
		message = '';

		const trimmedText = text.trim();
		if (!trimmedText) {
			message = 'Answer text cannot be empty';
			return;
		}

		try {
			const response = await fetch(`${API_BASE_URL}/answers/`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({ text, question_id: parseInt(question_id) })
			});
			if (response.ok) {
				message = 'Answer submitted successfully!';
				text = '';
			} else {
				const errorData = await response.text();
				message = `Failed to submit answer: ${errorData}`;
			}
		} catch (error) {
			message = 'Error submitting answer.';
		}
	}

	async function fetchAnswers() {
		loading = true;
		error = '';
		try {
			const res = await fetch(`${API_BASE_URL}/question/${question_id}/answers`);
			if (res.ok) {
				answers = await res.json();
			} else {
				error = 'Failed to load answers.';
			}
		} catch (e) {
			error = 'Error loading data. Please check your connection.';
		} finally {
			loading = false;
		}
	}
	onMount(() => {
		fetchAnswers();
	});

	$: if (message === 'Answer submitted successfully!') {
		fetchAnswers();
	}
</script>


<form on:submit|preventDefault={handleSubmit}>
	<label for="answer">Answer:</label>
	<input id="answer" type="text" bind:value={text} required />
	<button type="submit">Submit</button>
</form>

{#if message}
	<p>{message}</p>
{/if}

{#if loading}
	<p>Loading...</p>
{:else if error}
	<p>{error}</p>
{:else}
	<ul>
		{#each answers as answer}
			<li>{answer.text}</li>
		{/each}
	</ul>
{/if}