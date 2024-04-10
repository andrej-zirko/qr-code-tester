<script>
	import QRCode from "qrcode";
	import { onMount } from "svelte";
	import * as ed from "@noble/ed25519";
	import { encode } from "universal-base64url";
	import pako from "pako";
	import VcqrCode from "./VCQRCode.svelte";
	import { encode as base45encode } from "base45-js";

	let jsonInput = JSON.stringify({
		iss: "did:key:z6MktMmaoQuQLz1YN2p13yREKDAZRecPKPkmJXDDpvjN5sbo",
		sub: "did:key:z6MkioQsxkXo9NCTHaFq1UAg9mfC5C1JCLCq8x88gwvjfRT7#z6MkioQsxkXo9NCTHaFq1UAg9mfC5C1JCLCq8x88gwvjfRT7",
		vc: {
			"@context": [
				"https://www.w3.org/2018/credentials/v1",
				"https://my.sample-birthcertificate.eu",
			],
			id: "urn:uuid:229bf120-0f90-4cde-9ad6-15f207ee9432",
			type: ["VerifiableCredential", "BirthCertificate"],
			name: "Birth Certificate name",
			issuanceDate: "2024-04-09T08:04:47.502876726Z",
			expirationDate: "2025-04-09T08:04:47.502953643Z",
			credentialSubject: {
				id: "did:key:z6MkioQsxkXo9NCTHaFq1UAg9mfC5C1JCLCq8x88gwvjfRT7#z6MkioQsxkXo9NCTHaFq1UAg9mfC5C1JCLCq8x88gwvjfRT7",
				childsForename: "Saoirsebriginmuiregilliansiobhancaoimhemollygrainn",
				childsSurname: "Parent1OhAilpinOCiaranODonnchadhaODriscollOhUiginn",
				dateOfBirth: "1990-05-14",
				Parent1Forename:
					"Parent1Saoirsebriginmuiregilliansiobhancaoimhemoll",
				Parent1Surname:
					"Parent1Saoirsebriginmuiregilliansiobhancaoimhemoll",
				Parent2Forename:
					"Parent1Saoirsebriginmuiregilliansiobhancaoimhemoll",
				Parent2Surname:
					"Parent1Saoirsebriginmuiregilliansiobhancaoimhemoll"
			},
			issuer: {
				id: "did:key:z6MktMmaoQuQLz1YN2p13yREKDAZRecPKPkmJXDDpvjN5sbo",
			},
		},
		jti: "urn:uuid:229bf120-0f90-4cde-9ad6-15f207ee9432",
		exp: 1744185887,
		iat: 1712649887,
		nbf: 1712649887,
	});

	let isValidJson = true; // Flag to track if the input is valid JSON

	let vcQR = {
		token: "",
		data: "",
		errorMessage: "",
	};

	let vcBase45QR = {
		token: "",
		data: "",
		errorMessage: "",
	};

	let compressedVcQR = {
		token: "",
		data: "",
		errorMessage: "",
	};

	// Function to validate the JSON input
	const validateJson = () => {
		try {
			JSON.parse(jsonInput);
			isValidJson = true;
		} catch (error) {
			isValidJson = false;
		}
	};

	// Call the validateJson function whenever the jsonInput changes
	$: validateJson(jsonInput);

	// Auto-format the JSON input on mount
	onMount(() => {
		jsonInput = JSON.stringify(JSON.parse(jsonInput), null, 2);
	});

	const signJWT = async (payload, encodingFunction) => {
		const privateKey = ed.utils.randomPrivateKey();

		const header = {
			alg: "EdDSA",
			typ: "JWT",
			kid: "did:key:z6MktMmaoQuQLz1YN2p13yREKDAZRecPKPkmJXDDpvjN5sbo",
		};
		const encodedHeader = encodingFunction(JSON.stringify(header));
		const encodedPayload = encodingFunction(JSON.stringify(JSON.parse(payload)));

		const message = `${encodedHeader}.${encodedPayload}`;

		const signature = await ed.signAsync(
			Uint8Array.from(message),
			privateKey,
		);
		const encodedSignature = encodingFunction(
			String.fromCharCode.apply(null, new Uint8Array(signature)),
		);

		const token = `${message}.${encodedSignature}`;

		return token;
	};

	const compressJwtTokenGzip = (token) => {
		// Convert the token string to a byte array
		// const bytes = new TextEncoder().encode(token);

		// Compress the byte array using pako gzip
		const compressed = pako.gzip(token, { level: 9});

		// Convert the compressed byte array back to a string
		const compressedToken = String.fromCharCode.apply(
			null,
			new Uint8Array(compressed),
		);

		const compressedTokenBase64 = btoa(compressedToken);

		return compressedTokenBase64;
	};

	const generateQRCode = async (token) => {
		try {
			const segs = [{ data: token, mode: "auto" }];
			const data = await QRCode.toDataURL(segs, {
				errorCorrectionLevel: "H",
			});
			return { data, token, errorMessage: "" };
		} catch (err) {
			const errorMessage = `${err.message}. The token length is ${token.length}.`;
			return { data: "", token: "", errorMessage };
		}
	};

	const issueCredential = async () => {
		const token = await signJWT(jsonInput, encode);
		const tokenBase45 = await signJWT(jsonInput, base45encode);
		const compressedToken = compressJwtTokenGzip(token);

		vcQR = await generateQRCode(token);
		vcBase45QR = await generateQRCode(tokenBase45);
		compressedVcQR = await generateQRCode(compressedToken);
	};
</script>

<main>
	<h1>QR Code Size Tester</h1>
	<textarea
		bind:value={jsonInput}
		rows="20"
		cols="100"
	/>
	<p class:error={!isValidJson}>
		{isValidJson ? "Valid JSON" : "Invalid JSON"}
	</p>

	<div class="button-container">
		<button on:click={issueCredential}>Issue Credential</button>
	</div>
	<VcqrCode
		data={vcQR.data}
		token={vcQR.token}
		errorMessage={vcQR.errorMessage}
		label="Original token"
	/>
	<!-- <VcqrCode
		data={vcBase45QR.data}
		token={vcBase45QR.token}
		errorMessage={vcBase45QR.errorMessage}
		label="Base45 original token"
	/> -->
	<VcqrCode
		data={compressedVcQR.data}
		token={compressedVcQR.token}
		errorMessage={compressedVcQR.errorMessage}
		label="Compressed token"
	/>
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

	h2 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 2em;
		font-weight: 100;
	}

	.button-container {
		/* Added */
		margin-bottom: 1rem; /* Added */
	}

	.error {
		color: red;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
