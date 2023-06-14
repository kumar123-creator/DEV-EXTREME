<script>
  import { onMount } from "svelte";
  import "bootstrap/dist/css/bootstrap.min.css";
  import DevExpress from "devextreme";

  let jsonData = [];
  let gridData = [];
  let isCVUploadPopupVisible = false;
  let selectedRowData = null;
  let selectedCVId = null;

  async function uploadCV(file) {
    // Upload logic for the CV file
    console.log("Uploading CV file", file);
    // You can use fetch or any other method to upload the file to your API
  }

  function handleSave(e) {
    // Save logic for the row changes
    console.log("Saving row changes", e);
  }

  function handleClose() {
    isCVUploadPopupVisible = false;
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
      { dataField: "id", caption: "ID", width: 100 },
      { dataField: "firstName", caption: "First Name", width: 150 },
      { dataField: "surname", caption: "Surname", width: 150 },
      { dataField: "email", caption: "Email", width: 200 },
      { dataField: "mobile", caption: "Mobile", width: 150 },
      {
        caption: "Actions",
        width: 350,
        cellTemplate: function (container, options) {
          const cvUploadButton = document.createElement("button");
          cvUploadButton.innerText = "CV Upload";
          cvUploadButton.classList.add("btn", "btn-primary", "mr-2");
          cvUploadButton.addEventListener("click", function () {
            const rowData = options.data;
            selectedRowData = rowData;
            selectedCVId = rowData.cvId; // Assuming cvId is the property containing the CV file ID
            isCVUploadPopupVisible = true;
          });

          container.appendChild(cvUploadButton);

          const downloadButton = document.createElement("button");
          downloadButton.innerText = "Download CV";
          downloadButton.classList.add("btn", "btn-primary", "mr-2");
          downloadButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cvId = cvData.internal.cloudFile.id;
              const downloadLink = `https://api.recruitly.io/api/cloudfile/download?cloudFileId=${cvId}&apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77`;
              window.open(downloadLink);
            } else {
              alert("Failed to fetch CV file.");
            }
          });

          container.appendChild(downloadButton);

          const viewButton = document.createElement("button");
          viewButton.innerText = "View CV";
          viewButton.classList.add("btn", "btn-primary");
          viewButton.addEventListener("click", async () => {
            const cvResponse = await fetch(
              `https://api.recruitly.io/api/candidatecv/${options.data.id}?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA`
            );
            if (cvResponse.ok) {
              const cvData = await cvResponse.json();
              const cvHtml = cvData.html;
              if (cvHtml) {
                const cvWindow = window.open("", "_blank");
                cvWindow.document.write(cvHtml);
                cvWindow.document.close();
              } else {
                alert("CV file not found.");
              }
            } else {
              alert("Failed to fetch CV file.");
            }
          });

          container.appendChild(viewButton);
        },
        width: 250,
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
            items: [
              {
                itemType: "group",
                items: [
                  {
                    dataField: "firstName",
                    editorOptions: { width: 200 },
                  },
                  {
                    dataField: "surname",
                    editorOptions: { width: 200 },
                  },
                  {
                    dataField: "email",
                    editorOptions: { width: 300 },
                  },
                  {
                    dataField: "mobile",
                    editorOptions: { width: 150 },
                  },
                ],
              },
            ],
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
          onEditingStart: function (e) {
            const rowData = e.data;
            selectedRowData = rowData;
          },
          onInitNewRow: function (e) {
            const newRowData = e.data;
            // Initialize new row data if needed
          },
          onRowInserting: handleSave,
          onRowUpdating: handleSave,
          onRowRemoving: function (e) {
            // Remove logic for the row
            console.log("Removing row", e);
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

  .popup-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .popup-content {
    background-color: #fff;
    padding: 20px;
    border-radius: 5px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    max-width: 400px;
    width: 100%;
  }

  .popup-content h3 {
    color: #333;
    font-size: 18px;
    margin-bottom: 10px;
  }

  .popup-content input[type="file"] {
    margin-bottom: 10px;
  }

  .btn-primary {
    background-color: #007bff;
    color: #fff;
  }

  .btn-primary:hover {
    background-color: #0069d9;
  }

  .btn-secondary {
    background-color: #6c757d;
    color: #fff;
  }

  .btn-secondary:hover {
    background-color: #5a6268;
  }

  .mr-2 {
    margin-right: 10px;
  }
</style>

<div id="dataGrid"></div>

{#if isCVUploadPopupVisible}
  <div class="popup-overlay">
    <div class="popup-content">
      <h3>Upload CV</h3>
      <input type="file" id="cvFile" name="cvFile" accept=".pdf,.doc,.docx" />
      <button class="btn btn-primary" on:click={() => handleClose()}>
        Close
      </button>
    </div>
  </div>
{/if}
