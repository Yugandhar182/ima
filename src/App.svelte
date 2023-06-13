<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let selectedCandidateCV = null;

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
          button.innerText = "View CV";
          button.className = "btn btn-primary";
          button.addEventListener("click", function () {
            viewCV(options.data.id);
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

  async function viewCV(candidateId) {
    try {
      const response = await fetch(
        `https://api.recruitly.io/api/candidatecv/${candidateId}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
      );
      if (response.ok) {
        const cvData = await response.json();
        selectedCandidateCV = cvData;
        openCVModal();
      } else {
        throw new Error("Failed to fetch CV file");
      }
    } catch (error) {
      console.error(error);
    }
  }

  function openCVModal() {
    const modalElement = document.getElementById("cvModal");
    const modalBody = modalElement.querySelector(".modal-body");
    modalBody.innerText = JSON.stringify(selectedCandidateCV, null, 2);
    modalElement.classList.add("show");
    modalElement.style.display = "block";
  }

  function closeCVModal() {
    const modalElement = document.getElementById("cvModal");
    modalElement.classList.remove("show");
    modalElement.style.display = "none";
  }
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color:blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>

<div id="cvModal" class="modal fade">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">CV Information</h5>
        <button
          type="button"
          class="btn-close"
          aria-label="Close"
          onclick={() => closeCVModal()}
        ></button>
      </div>
      <div class="modal-body"></div>
    </div>
  </div>
</div>
