<script>
	import { onMount, afterUpdate } from "svelte";
	import 'bootstrap/dist/css/bootstrap.min.css';
	let images = [];
	let filteredImages = [];
	let currentPage = 1;
	let pageSize = 9;
	let totalPages = 1;
	let searchQuery = "";
	let selectedImage = null;
	let selectedFile = null; 
	let isUploadPopupOpen = false;
	let sortAsc = true;
  
	// Fetch the images when the component is mounted
	onMount(async () => {
	  try {
		const response = await fetch("https://picsum.photos/v2/list");
		if (response.ok) {
		  images = await response.json();
		  filteredImages = images;
		  console.log(images);
		  totalPages = Math.ceil(filteredImages.length / pageSize);
		} else {
		  console.error("Failed to fetch images:", response.status, response.statusText);
		}
	  } catch (error) {
		console.error("Error fetching images:", error);
	  }
	});
  
	// Update the totalPages whenever the images or pageSize change
	afterUpdate(() => {
	  totalPages = Math.ceil(filteredImages.length / pageSize);
	});
  
	function goToPage(page) {
	  currentPage = page;
	}
  
	function searchImages() {
	  filteredImages = images.filter((image) =>
		image.author.toLowerCase().includes(searchQuery.toLowerCase())
	  );
	  currentPage = 1;
	  totalPages = Math.ceil(filteredImages.length / pageSize);
	}
	function openUploadPopup() {
    isUploadPopupOpen = true;
  }

  function closeUploadPopup() {
    isUploadPopupOpen = false;
  }

  function handleFileChange(event) {
    const file = event.target.files[0];
    if (file) {
      selectedFile = URL.createObjectURL(file); // Store the selected file path
      closeUploadPopup();
    }
  }
  function sortData() {
  if (sortAsc) {
    filteredImages = images.sort((a, b) => a.author.localeCompare(b.author));
  } else {
    filteredImages = images.sort((a, b) => b.author.localeCompare(a.author));
  }
}

function toggleSortOrder() {
  sortAsc = !sortAsc;
  sortData();
}
$: filteredImages = images.filter((image) =>
  image.author.toLowerCase().includes(searchQuery.toLowerCase())
);

  </script>
  
  <main>
	<h1 style="color: crimson;">Image Gallery</h1>
  

	<div class="search-upload-container">
		<div class="search-container">
		  <input
			type="text"
			bind:value={searchQuery}
			placeholder="Search by author"
			on:input={searchImages}
		  />
		  <button on:click={searchImages}>Search</button>
		</div>
		<button class="btn btn-primary" on:click={toggleSortOrder}>
			Sort
		  </button>
		<div class="upload-container">
		  <button  style="background-color: yellow;" class="upload-button" on:click={openUploadPopup}>
			Upload Images
		  </button>
		</div>
	  </div>
	
	  {#if isUploadPopupOpen}
	  <div class="upload-popup">
		<div class="upload-popup-content">
		  <h2>Select Image File</h2>
		  <input
			type="file"
			id="file-input"
			accept="image/*"
			on:change={handleFileChange}
		  />
		  <button on:click={closeUploadPopup}>Cancel</button>
		</div>
	  </div>
	  {/if}
  
	{#if images.length > 0}
	  <div class="grid-container">
		{#each filteredImages.slice((currentPage - 1) * pageSize, currentPage * pageSize) as image, index}
		  <div class="grid-item">
			<img src={image.download_url} alt={image.author} style="width: 200px; height: 150px;" on:click={() => selectedImage = image}>
  
			<p>{image.author}</p>
		  </div>
		{/each}
	  </div>
	  <nav aria-label="Pagination">
		<ul class="pagination">
		  {#each Array.from({ length: totalPages }, (_, i) => i + 1) as page}
			<li class="page-item {currentPage === page ? 'active' : ''}">
			  <button class="page-link" on:click={() => goToPage(page)}>{page}</button>
			</li>
		  {/each}
		</ul>
	  </nav>
	{:else}
	  <p>Loading images...</p>
	{/if}
  
	{#if selectedImage}
	  <div class="modal">
		<div class="modal-content">
		  <button class="close" on:click={() => selectedImage = null}>Close</button>
		  <img src={selectedImage.download_url} alt={selectedImage.author} style="width: 900px; height: 550px;">
		  <p>{selectedImage.author}</p>
		</div>
	  </div>
	{/if}
  </main>
  <style>
	.grid-container {
	  display: grid;
	  grid-template-columns: repeat(3, 1fr);
	  gap: 20px;
	}
  
	.grid-item {
	  text-align: center;
	  border: 1px solid #ddd;
	  padding: 10px;
	  border-radius: 4px;
	}
  
	.pagination {
	  display: flex;
	  justify-content: center;
	  margin-top: 20px;
	}
  
	.pagination .page-item {
	  list-style: none;
	  margin-right: 5px;
	}
  
	.pagination .page-item .page-link {
	  padding: 5px 10px;
	  border: 1px solid #ddd;
	  border-radius: 4px;
	  cursor: pointer;
	}
  
	.pagination .page-item .page-link:focus,
	.pagination .page-item .page-link:hover {
	  background-color: #f2f2f2;
	}
  
	.pagination .page-item.active .page-link {
	  background-color: #007bff;
	  color: #fff;
	  border-color: #007bff;
	}
  
	.search-container {
	  display: flex;
	  align-items: center;
	  justify-content: flex-end;
	  margin-bottom: 20px;
	}
  
	.search-container input {
	  padding: 5px;
	  margin-right: 10px;
	  border: 1px solid #ddd;
	  border-radius: 4px;
	}
  
	.search-container button {
	  padding: 5px 10px;
	  border: 1px solid #ddd;
	  border-radius: 4px;
	  cursor: pointer;
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
	
    max-width: 800px;
  
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

  .close:hover {
    color: #000;
  }
 
  

  .close-button {
    margin-top: 5px;
    padding: 4px 10px;
    border: 1px solid #dd0909;
    border-radius: 4px;
    cursor: pointer;
  }


  .search-upload-container {
	  display: flex;
	  align-items: center;
	  justify-content: space-between;
	  margin-bottom: 20px;
	}

.upload-popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.upload-popup-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 4px;
  max-width: 400px;
  text-align: center;
}

.upload-popup h2 {
  margin-bottom: 10px;
}

.upload-popup input[type="file"] {
  margin-bottom: 20px;
}

.upload-popup button {
  padding: 10px 20px;
  background-color: #e8e1ee;
  color: #5c2d96;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}


  </style>
  