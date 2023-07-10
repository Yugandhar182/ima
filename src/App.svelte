<script>
	const API_KEY = '548841973896839';
	const API_SECRET = 'SMPrJaeSvS2hlxwCABPwUf6dpkE';
	const CLOUD_NAME = 'dv6i5lgia';
  
	let images = [];
  
	async function fetchImages() {
	  const response = await fetch(`https://cors-anywhere.herokuapp.com/https://api.cloudinary.com/v1_1/${CLOUD_NAME}/resources/image/upload`, {
		headers: {
		  Authorization: `Basic ${btoa(`${API_KEY}:${API_SECRET}`)}`
		}
	  });
  
	  const data = await response.json();
	  images = data.resources;
	}
  
	fetchImages();
  </script>
  
  <main>
	{#each images as image}
	  <img src={image.url} alt={image.public_id} style="width: 200px; height: 150px;" />
	{/each}
  </main>
  