<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];

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
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`
            );
            if (cvInfoResponse.ok) {
              const cvInfoData = await cvInfoResponse.json();
              const cvid = cvInfoData.cvid;
              console.log(cvInfoData);
              if (cvid) {
                downloadCv(cvid);
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("Failed to fetch CV file information.");
            }
          };

          const downloadCv = async (cvid) => {
            const downloadUrl = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvid}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
            const cvResponse = await fetch(downloadUrl);
            if (cvResponse.ok) {
              const cvBlob = await cvResponse.blob();
              const cvUrl = URL.createObjectURL(cvBlob);
              const cvLink = document.createElement("a");
              cvLink.href = cvUrl;
              cvLink.download = "cv.pdf"; // Replace with the appropriate filename
              cvLink.click();
              URL.revokeObjectURL(cvUrl);
            } else {
              alert("Failed to fetch CV file.");
            }
          };

          const downloadButton = document.createElement("button");
          downloadButton.classList.add("btn", "btn-primary");
          downloadButton.innerText = "Download CV";
          downloadButton.addEventListener("click", getCvInfo);
          container.appendChild(downloadButton);
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

<h1 style="color: blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
