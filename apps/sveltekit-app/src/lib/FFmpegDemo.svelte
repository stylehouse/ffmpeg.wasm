<script lang="ts">
	import { FFmpeg } from '@ffmpeg/ffmpeg';
	// @ts-ignore
	// import {fetchFile} from '@ffmpeg/util';
	import {fetchFile} from '$lib/not-ffmpeg-util';
	import type { LogEvent } from '@ffmpeg/ffmpeg/dist/esm/types';

	let videoEl: HTMLVideoElement;

	const baseURL = 'https://unpkg.com/@ffmpeg/core-mt@0.12.6/dist/esm';
	const videoURL = 'https://raw.githubusercontent.com/ffmpegwasm/testdata/master/video-15s.avi';

	let message = 'Clicks Start';

	async function transcode() {
		// warn about problems that may occur...
		if (!crossOriginIsolated) {
			console.warn("crossOriginIsolated is false: configure Cross-Origin-(Opener|Embedder)-Policy")
		}
		if (typeof SharedArrayBuffer == 'undefined') {
			console.warn("SharedArrayBuffer is false: is crossOriginIsolated false?")
		}
		const ffmpeg = new FFmpeg();
		message = 'Loading ffmpeg-core.js';
		ffmpeg.on('log', ({ message: msg }: LogEvent) => {
			message = msg;
			console.log(message);
		});
		await ffmpeg.load({
			coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript'),
			wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
			workerURL: await toBlobURL(`${baseURL}/ffmpeg-core.worker.js`, 'text/javascript')
		});
		message = 'Start transcoding';
		await ffmpeg.writeFile('test.avi', await fetchFile(videoURL));
		
		await ffmpeg.exec(['-i', 'test.avi', 'test.mp4']);

		message = 'Complete transcoding';


		const data = await ffmpeg.readFile('test.mp4');
		videoEl.src = URL.createObjectURL(
			new Blob([(data as Uint8Array).buffer], { type: 'video/mp4' })
		);

	}
	
    async function toBlobURL(url, mimeType) {
        try {
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP error ${response.status}`);
            }
            const blob = await response.arrayBuffer();
            return URL.createObjectURL(new Blob([blob], { type: mimeType }));
        } catch (err) {
            console.error(`Error loading ${url}: ${err.message}`);
            throw err;
        }
    }
</script>

<div>
	<!-- svelte-ignore a11y-media-has-caption -->
	<video bind:this={videoEl} controls />
	<br />
	<button on:click={() => transcode() }>Start</button>
	<p>{message}</p>
</div>
