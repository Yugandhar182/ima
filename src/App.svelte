<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";
  import { PDFDocument } from "pdf-lib";

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
      // Add the file button column
      {
        caption: "Actions",
        width: 100,
        cellTemplate: (container, options) => {
          console.log(cvText);
          const link = document.createElement("a");
          link.href = "#";
          link.innerHTML = "Download CV";
          link.className = "btn btn-primary";
          container.appendChild(link);

          link.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            const cvText = await cvResponse.text();

            // Create a new PDF document
            const pdfDoc = await PDFDocument.create();
            const page = pdfDoc.addPage();

            // Embed the CV text into the PDF document
            page.drawText(cvText, {
              x: 50,
              y: page.getHeight() - 50,
              size: 12,
              maxWidth: 500,
              maxHeight: 500,
              font: await pdfDoc.embedFont("Helvetica"),
              color: rgb(0, 0, 0),
            });

            // Generate the PDF file
            const pdfBytes = await pdfDoc.save();

            // Create a Blob from the PDF bytes
            const pdfBlob = new Blob([pdfBytes], { type: "application/pdf" });

            // Create a URL for the Blob
            const url = window.URL.createObjectURL(pdfBlob);

            // Create a temporary link element to trigger the download
            const a = document.createElement("a");
            a.href = url;
            a.download = "CV.pdf";
            a.click();

            // Revoke the URL to release the object URL
            window.URL.revokeObjectURL(url);
          });
        },
      },
      // Define other columns as needed
    ];

    const dataGrid = new DevExpress.ui.dxDataGrid(document.getElementById("dataGrid"), {
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
          confirmDeleteMessage: "Are you sure you want to delete this record?",
        },
      },
      paging: {
        pageSize: 10,
      },
      onInitialized: () => {
        // Any additional initialization code
      },
    });
  });
</script>

<style>
  #dataGrid {
    height: 400px;
  }
</style>

<h1 style="color:blue;">Job Candidate Details</h1>

<div id="dataGrid"></div>
