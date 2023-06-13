<script>
  import { onMount } from "svelte";
  import { createEventDispatcher } from "svelte";

  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let selectedCandidate = null;
  let cvHtml = null;

  const dispatch = createEventDispatcher();

  async function openPopup(candidate) {
    selectedCandidate = candidate;
    const cvResponse = await fetch(
      `https://api.recruitly.io/api/candidatecv/${candidate.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
    );
    if (cvResponse.ok) {
      const cvData = await cvResponse.json();
      cvHtml = cvData.html;
    } else {
      cvHtml = null;
      alert("Failed to fetch CV file.");
    }
    dispatch("openPopup");
  }

  onMount(async () => {
    const response = await fetch(
      "https://api.recruitly.io/api/candidate?apiKey=TEST9349C0221517DA4942E39B5DF18C68CDA154"
    );
    const responseData = await response.json();
    jsonData = responseData.data;

    gridData = jsonData.map((item) => ({
      id: item.id,
      firstName: item.firstName,
      surname: item.surname,
      email: item.email,
      mobile: item.mobile,
    }));

    const columns = [
      { dataField: "id", caption: "ID", width: 250 },
      { dataField: "firstName", caption: "Full Name", width: 200 },
      { dataField: "surname", caption: "Surname", width: 200 },
      { dataField: "email", caption: "Email", width: 200 },
      { dataField: "mobile", caption: "Mobile", width: 150 },
      {
        caption: "Actions",
        cellTemplate: function (container, options) {
          const button = document.createElement("button");
          button.className = "btn btn-primary";
          button.innerText = "View CV";
          button.addEventListener("click", () => {
            openPopup(options.data);
          });
          container.appendChild(button);
        },
        width: 150,
      },
      // Add other columns as needed
    ];

    const dataGrid = new DevExpress.ui.dxDataGrid(
      document.getElementById("dataGrid"),
      {
        dataSource: gridData,
        columns: columns,
        showBorders: true,
        filterRow: {
          visible: true,
        },
        editing: {
          allowDeleting: true,
          allowAdding: true,
          allowUpdating: true,
          mode: "popup",
          form: {
            labelLocation: "top",
          },
          popup: {
            showTitle: true,
            title: "Row in the editing state",
          },
          texts: {
            saveRowChanges: "Save",
            cancelRowChanges: "Cancel",
            deleteRow: "Delete",
            confirmDeleteMessage:
              "Are you sure you want to delete this record?",
          },
        },
        paging: {
          pageSize: 10,
        },

        onInitialized: () => {},
      }
    );
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color:blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>

<!-- Popup -->
{#if selectedCandidate}
  <div class="modal fade show" style="display: block; background-color: rgba(0, 0, 0, 0.5);">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">CV Details</h5>
          <button type="button" class="close" on:click={() => selectedCandidate = null; cvHtml = null;}}>
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          {#if cvHtml}
            <div id="cvContent" innerHTML={cvHtml}></div>
          {:else}
            <p>CV file not found.</p>
          {/if}
        </div>
      </div>
    </div>
  </div>
{/if}
