<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let cvPopupVisible = false;
  let cvHtml = "";

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
          const getCvInfo = async () => {
            const cvInfoResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvInfoResponse.ok) {
              const cvInfoData = await cvInfoResponse.json();
              const cvHtmlData = cvInfoData.html;
              if (cvHtmlData) {
                cvHtml = cvHtmlData;
                cvPopupVisible = true;
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("Failed to fetch CV file information.");
            }
          };

          const downloadCv = async () => {
            const cvDownloadLink = document.createElement("a");
            cvDownloadLink.href = `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`;
            cvDownloadLink.target = "_blank";
            cvDownloadLink.click();
          };

          const downloadButton = document.createElement("button");
          downloadButton.classList.add("btn", "btn-primary", "mr-2");
          downloadButton.innerText = "Download CV";
          downloadButton.addEventListener("click", downloadCv);
          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
          viewButton.classList.add("btn", "btn-primary");
          viewButton.innerText = "View CV";
          viewButton.addEventListener("click", getCvInfo);
          container.appendChild(viewButton);
        },
        width: 200,
      },
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

<h1 style="color: blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>

<div
  class="modal fade"
  tabindex="-1"
  role="dialog"
  style="display: {cvPopupVisible ? 'block' : 'none'}"
>
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">CV Information</h5>
        <button
          type="button"
          class="close"
          data-dismiss="modal"
          aria-label="Close"
          on:click={() => (cvPopupVisible = false)}
        >
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        {@html cvHtml}
      </div>
      <div class="modal-footer">
        <button
          type="button"
          class="btn btn-secondary"
          data-dismiss="modal"
          on:click={() => (cvPopupVisible = false)}
        >
          Close
        </button>
      </div>
    </div>
  </div>
</div>
