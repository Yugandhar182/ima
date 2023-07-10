
<script>
	import { onMount } from 'svelte';
  
	let myWidget;
	let images = [];
  
	const CLOUDINARY_CLOUD_NAME = 'dv6i5lgia';
	const CLOUDINARY_API_KEY = '548841973896839';
	const CLOUDINARY_API_SECRET = 'SMPrJaeSvS2hlxwCABPwUf6dpkE';
  
	function openWidget() {
	  myWidget.open();
	}
  
	onMount(() => {
	  const script = document.createElement('script');
	  script.src = 'https://upload-widget.cloudinary.com/global/all.js';
	  script.async = true;
	  script.onload = () => {
		myWidget = cloudinary.createUploadWidget(
		  {
			cloudName: CLOUDINARY_CLOUD_NAME,
			uploadPreset: 'csniru89' // Replace with your upload preset name
		  },
		  (error, result) => {
			if (!error && result && result.event === 'success') {
			  console.log('Done! Here is the image info: ', result.info);
			  fetchImages();
			}
		  }
		);
	  };
	  document.body.appendChild(script);
  
	  fetchImages();
	});
  
	async function fetchImages() {
		const response = await fetch(`https://cors-anywhere.herokuapp.com/https://api.cloudinary.com/v1_1/${CLOUDINARY_CLOUD_NAME}/resources/image/upload`, {
		  headers: {
			Authorization: `Basic ${btoa(`${CLOUDINARY_API_KEY}:${ CLOUDINARY_API_SECRET}`)}`
		  }
		});
	
		const data = await response.json();
		images = data.resources;
	  }
	
	  fetchImages();
  </script>
  
  <style>
	.cloudinary-button {
	  margin: 10px;
	  padding: 10px 20px;
	  background-color: #4CAF50;
	  color: white;
	  border: none;
	  border-radius: 4px;
	  cursor: pointer;
	}
  
	.image-container {
	  display: flex;
	  flex-wrap: wrap;
	  margin-top: 20px;
	}
  
	.image-item {
	  width: 200px;
	  margin: 10px;
	  overflow: hidden;
	}
  
	.image-item img {
	  width: 100%;
	  height: auto;
	}
  </style>
  
  <div class="image-container">
	{#each images as image}
		<img src={image.url} alt={image.public_id} style="width: 200px; height: 150px;" />
	  {/each}
  </div>
  
  <button id="upload_widget" class="cloudinary-button" on:click={openWidget}>
	Upload files
  </button>
  