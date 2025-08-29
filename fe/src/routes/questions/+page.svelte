<script lang="ts">
	import { onMount } from 'svelte';
	
	const API_BASE_URL = 'http://localhost:8000';

	let questions: Question[] = [];
	let loading = true;
	let error = '';
	let text = '';
	let message = '';

	async function handleSubmit(event: SubmitEvent) {
		event.preventDefault();
		message = '';
		
		const trimmedText = text.trim();
		if (!trimmedText) {
			message = 'Question text cannot be empty';
			return;
		}

		try {
			const response = await fetch(`${API_BASE_URL}/questions/`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({ text })
			});
			if (response.ok) {
				message = 'Question submitted successfully!';
				text = '';
			} else {
				const errorData = await response.text();
				message = `Failed to submit question: ${errorData}`;
			}
		} catch (error) {
			message = 'Error submitting question. Please check your connection.';
		}
	}

	async function fetchQuestions() {
		loading = true;
		error = '';
		try {
			const res = await fetch(`${API_BASE_URL}/questions/`);
			if (res.ok) {
				questions = await res.json();
			} else {
				error = 'Failed to load questions.';
			}
		} catch (e) {
			error = 'Error loading questions.';
		} finally {
			loading = false;
		}
	}

	onMount(fetchQuestions);

	//Refresh questions after submitting a new one
	$: if (message === 'Question submitted successfully!') {
		fetchQuestions();
	}


</script>

<form on:submit|preventDefault={handleSubmit}>
	<label for="question">Question:</label>
	<input id="question" type="text" bind:value={text} required />
	<button type="submit">Submit</button>
</form>

{#if message}
	<p>{message}</p>
{/if}


{#if loading}
	<p>Loading questions...</p>
{:else if error}
	<p>{error}</p>
{:else}
	<h2>All Questions</h2>
	<ul>
		{#each questions as question}
			<li>{question.text}</li>
			<a href="/questions/{question.id}/answers">Show Answers</a>
		{/each}
	</ul>
{/if}