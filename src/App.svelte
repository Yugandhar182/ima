<script>
  import { onMount , afterUpdate } from 'svelte';
  import 'bootstrap/dist/css/bootstrap.min.css';

  let myWidget;
  let images = [];
  let currentPage = 1;
  const itemsPerPage = 9;
  let filteredImages = [];
  let searchText = '';
  let searchTimeout;
  let selectedImage = null;
  let sortAsc = true;
   let filterHeightAsc = true;
  let filterWidthAsc = true;
  let filterOption = "";

  const CLOUDINARY_CLOUD_NAME = 'dv6i5lgia';
  const CLOUDINARY_API_KEY = '548841973896839';
  const CLOUDINARY_API_SECRET = 'SMPrJaeSvS2hlxwCABPwUf6dpkE';

  function openWidget() {
    myWidget.open();
  }

  function handleSearchInput() {
    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(searchImages, 500); // Delayed search execution
  }

  function searchImages() {
    filteredImages = images.filter(image => image.public_id.startsWith(searchText));
    currentPage = 1;
    paginateImages();
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
	
    paginateImages();
  }

  function paginateImages() {
    if (images) {
      const startIndex = (currentPage - 1) * itemsPerPage;
      const endIndex = currentPage * itemsPerPage;
      const paginatedImages = searchText ? filteredImages : images;
      filteredImages = paginatedImages.slice(startIndex, endIndex);
    }
  }

  function goToPage(pageNumber) {
    currentPage = pageNumber;
    paginateImages();
  }

  fetchImages();

    function sortData() {
    if (sortAsc) {
      images = images.sort((a, b) => a.public_id.localeCompare(b.public_id));
    } else {
      images = images.sort((a, b) => b.public_id.localeCompare(a.public_id));
    }

    paginateImages();
  }
  function toggleSortOrder() {
    sortAsc = !sortAsc;
    sortData();
  }


   
    function filterByHeight() {
    filterOption = filterHeightAsc ? "ascHeight" : "descHeight";
    filterHeightAsc = !filterHeightAsc;
  }

  function filterByWidth() {
    filterOption = filterWidthAsc ? "ascWidth" : "descWidth";
    filterWidthAsc = !filterWidthAsc;
  }

  function applyFilter() {
    if (filterOption === "ascHeight") {
      filteredImages = images.slice().sort((a, b) => a.height - b.height);
    } else if (filterOption === "descHeight") {
      filteredImages = images.slice().sort((a, b) => b.height - a.height);
    } else if (filterOption === "ascWidth") {
      filteredImages = images.slice().sort((a, b) => a.width - b.width);
    } else if (filterOption === "descWidth") {
      filteredImages = images.slice().sort((a, b) => b.width - a.width);
    } else {
      // No filter option selected, display all images
      filteredImages = images;
    }
	
  }
</script>

<div>
  <input type="text" bind:value={searchText} placeholder="Search by Name" on:input={handleSearchInput} />
  <button on:click={searchImages}>Search</button>
   <span class="spacer"></span>
  <button class="btn btn-primary" on:click={toggleSortOrder}>
    Sort
  </button>
   <span class="spacer"></span>
  
  <div class="filter-container">
        <select bind:value={filterOption}>
          <option value="">No Filter</option>
          <option value="ascHeight">Asc Height</option>
          <option value="descHeight">Desc Height</option>
          <option value="ascWidth">Asc Width</option>
          <option value="descWidth">Desc Width</option>
        </select>
        <button on:click={applyFilter}>Filter</button>
      </div>
    </div>


<div class="image-container">
  {#each filteredImages as image}
  <div class="image-item">
    <img src={image.url} alt={image.public_id} style="width: 400px; height: 200px;" on:click={() => selectedImage = image} />
    <div class="image-name">{image.public_id}</div>
  </div>
  {/each}
</div>

<div class="pagination">
  {#each Array.from({ length: Math.ceil(images.length / itemsPerPage) }) as _, index}
  <div class="pagination-item {index + 1 === currentPage ? 'active' : ''}" on:click={() => goToPage(index + 1)}>
    {index + 1}
  </div>
  {/each}
</div>

<button id="upload_widget" class="cloudinary-button" on:click={openWidget}>
  Upload images
</button>

{#if selectedImage}
<div class="modal">
  <div class="modal-content">
    <button class="close" on:click={() => selectedImage = null}>Close</button>
    <img src={selectedImage.url} alt={selectedImage.public_id} style="width: 900px; height: 550px;" />
  </div>
</div>
{/if}

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
    margin-top: 50px;
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

  .modal {
    display: block;
    position: fixed;
    z-index: 999;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: hsl(0, 12%, 7%);
  }

  .modal-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    position: relative;
    margin: auto;
    margin-top: 10%;
    width: 80%;
    margin-top: 80px;
    max-width: 800px;
    background-color: hsl(0, 12%, 7%);
    padding: 20px;
    border-radius: 4px;
  }

  .close {
    position: absolute;
    top: 10px;
    right: 20px;
    font-size: 24px;
    font-weight: bold;
    color: #ea1e40;
    cursor: pointer;
  }
     .spacer {
  margin: 0 50px;
}

  .filter-container {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    margin-bottom: 20px;
	margin-left: 40px;
  }

  .filter-container select {
    margin-right: 10px;
	margin-left: 40px;
		margin-top: -40px;
  }

  .filter-container button {
    padding: 5px 10px;
    border: 1px solid #5b69eb;
    border-radius: 4px;
    cursor: pointer;
	margin-top: -40px;
	margin-right: 700px;
	background-color: hsl(329, 82%, 74%);
  }
</style>
