<script lang="ts">
	import { page } from '$app/stores';
	import { onMount } from 'svelte';

	let inferenceServer = 'http://localhost:19000/v1/completions';
	let query = '';
	let results = '';

	onMount(() => {});

	const getPrediction = () => {
		fetch(inferenceServer, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				prompt: query,
				temperature: 0.7,
				stream: false,
				max_tokens: 2048,
				stop: ['###', '\n']
			})
		})
			.then((res) => res.json())
			.then((data) => {
				//console.log(data);
				if (data.choices[0].text) {
					results = data.choices[0].text;
				} else {
					results = '';
				}
			});
	};

	const getPredictionStreaming = async () => {
		const response = await fetch(inferenceServer, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				prompt: query,
				temperature: 0.7,
				stream: true,
				max_tokens: 2048,
				stop: ['###', '\n']
			})
		});

		if (!response || !response.body) {
			return;
		}

		// Read the response as a stream of data
		const reader = response.body.getReader();
		const decoder = new TextDecoder('utf-8');
		results = '';

		while (true) {
			const { done, value } = await reader.read();
			if (done) {
				break;
			}

			const chunk = decoder.decode(value);
			const lines = chunk.split('\n');
			for (const line of lines) {
				const stripped = line.replace(/^data: /, '').trim();
				if (stripped == '') {
					continue;
				}
				if (stripped.endsWith('[DONE]')) {
					continue;
				}
				const parsed = JSON.parse(stripped);
				if (parsed.choices && parsed.choices[0] && parsed.choices[0].text) {
					results += parsed.choices[0].text;
				}
			}
		}
	};
</script>

<section class="p-4 w-[600px]">
	<div class="flex flex-col gap-4 w-full">
		<textarea bind:value={query} name="query" class="rounded-xl bg-slate-100 p-4 h-32 w-full" />
		<input
			type="button"
			value="Get Prediction"
			on:click={getPredictionStreaming}
			class="rounded-xl p-4 bg-slate-300 text-slate-500 hover:bg-slate-400"
		/>

		<textarea class="p-4 my-4 rounded-xl bg-slate-100 h-64" value={results} />
	</div>
</section>
