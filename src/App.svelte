<script>
	import { onMount } from 'svelte';
  
	const API_KEY = '548841973896839';
	const API_SECRET = 'SMPrJaeSvS2hlxwCABPwUf6dpkE';
	const CLOUD_NAME = 'dv6i5lgia';
  
	let images = [];
  
	async function fetchImages() {
	  try {
		const response = await fetch(`https://api.cloudinary.com/v1_1/${CLOUD_NAME}/resources/image`, {
		  headers: {
			'Authorization': 'Basic ' + btoa(`${API_KEY}:${API_SECRET}`)
		  }
		});
		const data = await response.json();
		images = data.resources;
		console.log('Fetched images:', images);
	  } catch (error) {
		console.error('Error fetching images:', error);
	  }
	}
  
	onMount(fetchImages);
  </script>
  
  <ul>
	{#each images as image}
	  <li>{image.public_id}</li>
	{/each}
  </ul>
  