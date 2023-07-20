<script lang="ts">
	import { onMount } from 'svelte';
	import { env } from '$env/dynamic/public';
	//	const inferenceServer = 'http://localhost:19000/v1/completions';
	type Config = {
		url: string;
		headers: Record<string, string>;
		params?: Record<string, string>;
	};

	const services: { [key: string]: Config } = {
		'local-llama': {
			url: 'http://localhost:19000/v1/completions',
			headers: {}
		},
		openai: {
			url: 'https://api.openai.com/v1/completions',
			headers: {
				Authorization: `Bearer ${env.PUBLIC_OPENAI_API_KEY}`
			},
			params: {
				model: 'text-davinci-003'
			}
		}
	};

	const config = services['local-llama'];
	let query = '';
	let results = '';

	onMount(() => {});

	const getPrediction = () => {
		fetch(config.url, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json',
				...config.headers
			},
			body: JSON.stringify({
				prompt: query,
				temperature: 0.7,
				stream: false,
				max_tokens: 512,
				stop: ['###']
			})
		})
			.then((res) => res.json())
			.then((data) => {
				if (data.choices[0].text) {
					results = data.choices[0].text;
				} else {
					results = '';
				}
			});
	};

	const getPredictionStreaming = async () => {
		let params = {
			prompt: query,
			temperature: 0.5,
			stream: true,
			max_tokens: 512,
			stop: ['###']
		};

		if (config.params) {
			params = { ...params, ...config.params };
		}

		const response = await fetch(config.url, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json',
				...config.headers
			},
			body: JSON.stringify(params)
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
				console.log('Data -->', stripped);
				if (stripped == '') {
					continue;
				}
				if (stripped.endsWith('[DONE]')) {
					break;
				}
				if (stripped.startsWith(': ping')) {
					// ignore
					// : ping - 2023-07-20 12:23:01.384429
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
