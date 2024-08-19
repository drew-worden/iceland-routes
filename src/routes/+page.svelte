<script>
	// Import images or use URLs
	import landmannalaugarImage from '$lib/assets/pexels-tomas-malik-793526-27245704.jpg';
    import hornstrandirImage from '$lib/assets/pexels-jonathanschmer-2364402.jpg';
</script>

<div class="landing-page">
	<div class="card">
		<img src={landmannalaugarImage} alt="Landmannalaugar" />
		<div class="card-title">Landmannalaugar</div>
		<a href="/landmannalaugar"></a>
	</div>
	<div class="card">
		<img src={hornstrandirImage} alt="Hornstrandir" />
		<div class="card-title">Hornstrandir</div>
		<a href="/hornstrandir"></a>
	</div>
</div>

<style>
	.landing-page {
		display: flex;
		gap: 64px;
		padding: 64px;
		height: 100vh;
		background-color: #f0f0f0;
		font-family: 'Arial', sans-serif;
	}

	.card {
		flex: 1;
		position: relative;
		border-radius: 10px;
		overflow: hidden;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
		transition: transform 0.1s ease-in-out;
	}

	.card:hover {
		transform: scale(1.05);
	}

	.card img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}

	.card-title {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		background-color: rgba(0, 0, 0, 0.5);
		color: white;
		padding: 20px;
		text-align: center;
		font-size: 1.5rem;
	}

	.card a {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		text-decoration: none;
		color: inherit;
	}
</style>
