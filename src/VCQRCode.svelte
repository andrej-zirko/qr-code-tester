<script>
    export let data; // The data to be displayed (e.g., qrCodeData or compressedQrCodeData)
    export let token; // The token to be displayed (e.g., token or compressedToken)
    export let errorMessage; // The error message to be displayed (e.g., errorMessage or compressedErrorMessage)
    export let label = 'Token'; // Optional label for the token
  </script>

{#if data}
<div>
    <h2>{label} (size {token.length})</h2>
    <p>{token}</p>
</div>
<img src={data} alt="QR code" />
{:else if errorMessage}
<p style="color: red;">{label} - {errorMessage}</p>
{/if}