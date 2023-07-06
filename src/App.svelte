<script>
  import { onMount } from "svelte";
  import { readable } from "svelte/store";

  // Sample data
  let jsonData = [];
  let data = [];

  const fetchImages = async () => {
    const response = await fetch("https://picsum.photos/v2/list");
    const responseData = await response.json();
    return responseData;
  };

  onMount(async () => {
    const images = await fetchImages();
    jsonData = images;

    const gridData = jsonData.map((item) => ({
      id: item.id,
      reference: item.author,
      name: item.filename,
      email: item.post_url,
      phone: item.download_url,
    }));

    const imageStore = readable(gridData, (set) => {
      set(gridData);
    });

    var dataGrid = new DevExpress.ui.dxDataGrid("#dataGrid", {
      dataSource: {
        store: imageStore,
        paginate: true,
        pageSize: 10,
      },
      columns: [
        { dataField: "id", caption: "ID" },
        { dataField: "reference", caption: "Author" },
        { dataField: "name", caption: "Filename" },
        { dataField: "email", caption: "Post URL" },
        { dataField: "phone", caption: "Download URL" },
      ],
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
      },
      paging: {
        pageSize: 10,
      },
      pager: {
        showPageSizeSelector: true,
        allowedPageSizes: [5, 10, 20],
        showInfo: true,
      },
    });
  });
</script>

<div id="dataGrid"></div>
