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
      // Add the file button column
      {
        caption: "Actions",
        width: 100,
        cellTemplate: (container, options) => {
          const link = document.createElement("a");
          link.href = "#";
          link.innerHTML = "Download CV";
          link.className = "btn btn-primary";
          container.appendChild(link);

          link.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/cloudfile/download?cloudFileId=2d969dc4-36d3-47b9-9d40-0351d1ab6f3a&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`
            );
            const cvString = await cvResponse.text();

            // Create a new Blob with the string content
            const blob = new Blob([cvString], { type: "text/plain" });

            // Create a temporary link element
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.download = "CV.txt";
            link.click();

            // Clean up the temporary link
            URL.revokeObjectURL(link.href);
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
          title: "Row in the editing ",
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
