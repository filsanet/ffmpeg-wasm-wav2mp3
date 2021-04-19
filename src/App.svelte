<script>
import 'chota';
import { Button, Card, Modal } from 'svelte-chota'
import Converter from  "./Converter.svelte";

let modal_open = false;
</script>

<main>
	<div class="spacer"></div>
	<section>

			<h1>Audio Converter</h1>
			<p>Convert WAV to MP3 <button class="info-btn" on:click={event => modal_open = true}>i</button></p>
			<Converter />
			
			<Modal bind:open={modal_open}>
				<Card>
					<h4 class="is-left">Info</h4>
					<p class="is-left">This converter works entirely in your browser. No data is uploaded to the servers.</p>
					<p class="is-left">Only converts one file at a time, .wav to .mp3 only.</p>

				</Card>
			</Modal>
	</section>
</main>
	
<style>
	main {
		background-color: #4f4f4f;
		margin: 0px auto;
		min-height: 100vh;
	}
	section {
		background-color: white;
		text-align: center;
		padding: 1em;
		max-width: 800px;
		min-height: 525px;
		margin: auto;
		border-radius: 50px;
	}

	h1 {
		color: #ca2420;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	.info-btn {
		border-color: rgb(81, 110, 172);
		color: rgb(81, 110, 172);
		border-radius: 50px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
	div.spacer {
		height: 2em;
	}


</style>