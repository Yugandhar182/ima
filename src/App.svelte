<script>
  import { onMount } from 'svelte';
  import 'bootstrap/dist/css/bootstrap.min.css';

  let myWidget;
  let images = [];
  let filteredImages = [];
  let searchInput = '';
  let currentPage = 1;
  const itemsPerPage = 9;

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
    const response = await fetch(
      `https://cors-anywhere.herokuapp.com/https://api.cloudinary.com/v1_1/${CLOUDINARY_CLOUD_NAME}/resources/image?max_results=100`,
      {
        headers: {
          Authorization: `Basic ${btoa(`${CLOUDINARY_API_KEY}:${CLOUDINARY_API_SECRET}`)}`
        }
      }
    );

    const data = await response.json();
    images = data.resources;
    filterImages();
  }

  function filterImages() {
    if (searchInput === '') {
      filteredImages = images;
    } else {
      filteredImages = images.filter(image => image.public_id.startsWith(searchInput));
    }

    paginateImages();
  }

  function paginateImages() {
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = currentPage * itemsPerPage;
    filteredImages = images.slice(startIndex, endIndex);
  }

  function goToPage(pageNumber) {
    currentPage = pageNumber;
    paginateImages();
  }

  fetchImages();
</script>

<style>
  .cloudinary-button {
    margin: 10px;
    padding: 10px 20px;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    position: absolute;
    top: 10px;
    right: 10px;
  }

  .image-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
   grid-gap: 10px;
    margin-top: 80px;
    margin-bottom: 20px; 
  }

  .image-item {
    width: 100%;
    height: auto;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    text-align: center;
  }

  .image-item img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .image-name {
    margin-top: 10px;
    font-size: 14px;
  }

  .search-container {
    position: absolute;
    top: 10px;
    left: 10px;
  }

  .search-input {
    padding: 5px;
  }

  .pagination {
    display: flex;
    justify-content: center;
    margin-top: 20px;
  }

  .pagination-item {
    margin: 0 5px;
    padding: 5px 10px;
    background-color: #f1f1f1;
    color: #333;
    cursor: pointer;
  }

  .pagination-item.active {
    background-color: #4caf50;
    color: #fff;
  }
</style>

<div class="search-container">
  <input type="text" class="search-input" bind:value={searchInput} placeholder="Search by name" on:input={filterImages} />
</div>

<div class="image-container">
  {#each filteredImages as image}
  <div class="image-item">
    <img src={image.url} alt={image.public_id} style="width: 400px; height: 200px;" />
    <div class="image-name">{image.public_id}</div>
  </div>
  {/each}
</div>

<div class="pagination">
  {#each Array.from({ length: Math.ceil(images.length / itemsPerPage) }) as _, index}
  <div
    class="pagination-item {index + 1 === currentPage ? 'active' : ''}"
    on:click={() => goToPage(index + 1)}
  >
    {index + 1}
  </div>
  {/each}
</div>

<button id="upload_widget" class="cloudinary-button" on:click={openWidget}>
  Upload files
</button>
