<!-- App.svelte -->
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

  function fetchImages() {
    const API_KEY = CLOUDINARY_API_KEY;
    const API_SECRET = CLOUDINARY_API_SECRET;
    const credentials = `${API_KEY}:${API_SECRET}`;
    const encodedCredentials = btoa(credentials);

    fetch(`https://api.cloudinary.com/v1_1/${CLOUDINARY_CLOUD_NAME}/resources/image`, {
      method: 'get',
      headers: {
        'Authorization': 'Basic ' + encodedCredentials,
      },
    })
    .then(res => res.json())
    .then(data => {
      images = data.resources;
    })
    .catch(error => {
      console.error('Error fetching images:', error);
    });
  }
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
    <div class="image-item">
      <img src={image.url} alt={image.public_id} />
    </div>
  {/each}
</div>

<button id="upload_widget" class="cloudinary-button" on:click={openWidget}>
  Upload files
</button>
