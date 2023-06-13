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
          const downloadLink = document.createElement("a");
          downloadLink.href = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${options.data.id}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
          downloadLink.target = "_blank";
          downloadLink.download = `CV_${options.data.id}.pdf`;
          downloadLink.innerText = "Download CV";
          downloadLink.addEventListener("click", async (event) => {
            event.preventDefault();
            const cvResponse = await fetch(downloadLink.href);
            if (cvResponse.ok) {
              const cvBlob = await cvResponse.blob();
              const cvUrl = URL.createObjectURL(cvBlob);
              const cvLink = document.createElement("a");
              cvLink.href = cvUrl;
              cvLink.download = downloadLink.download;
              cvLink.click();
              URL.revokeObjectURL(cvUrl);
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(downloadLink);

          const viewLink = document.createElement("a");
          viewLink.href = `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`;
          viewLink.target = "_blank";
          viewLink.innerText = "View CV";
          viewLink.addEventListener("click", async (event) => {
            event.preventDefault();
            const cvResponse = await fetch(viewLink.href);
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cvHtml = cvData.html;
              if (cvHtml) {
                const cvPopup = document.createElement("div");
                cvPopup.className = "cv-popup";

                const cvContent = document.createElement("div");
                cvContent.className = "cv-popup-content";
                cvPopup.appendChild(cvContent);

                const closeBtn = document.createElement("span");
                closeBtn.className = "cv-popup-close";
                closeBtn.innerText = "Close";
                closeBtn.addEventListener("click", () => {
                  cvPopup.remove();
                });
                cvContent.appendChild(closeBtn);

                const cvFrame = document.createElement("iframe");
                cvFrame.srcdoc = cvHtml;
                cvFrame.style.width = "100%";
                cvFrame.style.height = "100%";
                cvContent.appendChild(cvFrame);

                document.body.appendChild(cvPopup);
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("Failed to fetch CV file.");
            }
          });
          container.appendChild(viewLink);
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

  .cv-popup {
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

  .cv-popup-content {
    background-color: white;
    padding: 20px;
    max-width: 800px;
    max-height: 80%;
    overflow-y: auto;
    text-align: center;
    position: relative;
  }

  .cv-popup-close {
    position: absolute;
    top: 10px;
    right: 10px;
    cursor: pointer;
  }
</style>

<h1 style="color: blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
