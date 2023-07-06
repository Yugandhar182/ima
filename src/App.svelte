<script>
	import { onMount, afterUpdate } from "svelte";
  
	let images = [];
	let filteredImages = [];
	let currentPage = 1;
	let pageSize = 9;
	let totalPages = 1;
	let searchQuery = "";
  
	// Fetch the images when the component is mounted
	onMount(async () => {
	  try {
		const response = await fetch("https://picsum.photos/v2/list");
		if (response.ok) {
		  images = await response.json();
		  filteredImages = images;
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
</script>
  
<main>
	<div class="search-container">
	  <div class="search-box">
	    <input type="text" bind:value={searchQuery} placeholder="Search by author" />
	    <button on:click={searchImages}>Search</button>
	  </div>
	</div>
  
	{#if images.length > 0}
	  <div class="grid-container">
		{#each filteredImages.slice((currentPage - 1) * pageSize, currentPage * pageSize) as image, index}
		  <div class="grid-item">
			<img src={image.download_url} alt={image.author} style="width: 200px; height: 150px;">
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
</main>
  
<style>
	.grid-container {
	  display: grid;
	  grid-template-columns: repeat(3, 1fr);
	  gap: 20px;
	  width: 250px;
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
	  justify-content: flex-start; /* Updated alignment */
	  margin-bottom: 20px;
	  margin-left: 20px; /* Added margin to position search container */
	}
  
	.search-container .search-box {
	  display: flex;
	  align-items: center;
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
</style>
