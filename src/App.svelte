<script>
	export let name;

	import QRCode from "qrcode";

	let size = 100; // Default size
	let qrCodeData = "";
	let randomString = "";
	let errorMessage = ""; // Variable to store the error message

	function generateRandomString(length) {
		let result = "";
		const characters =
			"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
		const charactersLength = characters.length;
		for (let i = 0; i < length; i++) {
			result += characters.charAt(
				Math.floor(Math.random() * charactersLength),
			);
		}
		return result;
	}

	function generateQRCode() {
		randomString = generateRandomString(size);
		console.log(randomString);

		// alphanumeric
		const segs = [{ data: randomString, mode: "auto" }];

		QRCode.toDataURL(
			segs,
			{ errorCorrectionLevel: "H" },
			(err, url) => {
				if (err) {
					errorMessage = err.message; // Store the error message
					qrCodeData = ""; // Clear the QR code data
					randomString = "";
				} else {
					errorMessage = ""; // Clear the error message
					qrCodeData = url;
				}
			},
		);
	}
</script>

<main>
	<h1>QR Code Size Tester</h1>
	<label>
		String Size:
		<input type="number" bind:value={size} min="1" max="2000" />
	</label>
	<div class="button-container">
		<button on:click={generateQRCode}>Generate QR Code</button>
	</div>
	{#if qrCodeData}
		<img src={qrCodeData} alt={randomString} />
	{:else if errorMessage}
		<p style="color: red;">{errorMessage}</p>
		<!-- Display the error message in red -->
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	.button-container {
		/* Added */
		margin-bottom: 1rem; /* Added */
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
