<!-- <template>
  <div id="app">
    <input type="file" @change="loadCSV" />
    <button @click="generateTable">Generate Table</button>
    <div v-if="progress < 100">
      <div>Progress: {{ progress }}%</div>
      <progress :value="progress" max="100"></progress>
    </div>
    <div v-html="tableHtml"></div>
    <button @click="saveHtml">Save HTML</button>
  </div>
</template> -->

<template>
  <div id="app" style="padding: 20px; max-width: 800px; margin: left">
    <h2>CSV to HTML Table Converter</h2>
    <div style="margin-bottom: 20px">
      <input
        type="file"
        @change="loadCSV"
        style="display: block; margin-bottom: 10px" />
      <button
        v-if="csvRows.length > 0"
        @click="generateTable"
        style="
          padding: 10px 20px;
          border: none;
          background-color: #007bff;
          color: white;
          cursor: pointer;
        ">
        Generate Table
      </button>
      <button
        v-if="tableHtml"
        style="
          padding: 10px 20px;
          border: none;
          background-color: green;
          color: white;
          cursor: pointer;
          margin-left: 10px;
        "
        @click="saveHtml">
        Save HTML
      </button>
    </div>
    <div v-if="progress > 0 && progress < 100" style="margin-bottom: 20px">
      <div>Progress: {{ progress }}%</div>
      <progress
        :value="progress"
        max="100"
        style="width: 100%; height: 20px"></progress>
    </div>
    <div v-html="tableHtml" style="margin-top: 20px"></div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      tableHtml: '',
      csvRows: [],
      progress: 0,
    };
  },
  methods: {
    async loadCSV(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const csv = e.target.result;
          const rows = csv.split('\n').map((row) => row.split(','));
          this.csvRows = rows;
        };
        reader.readAsText(file);
      }
    },
    async generateTable() {
      // Assuming the first row contains headers
      let html = '<table border="1"><tr>';
      this.csvRows[0].forEach((header) => {
        html += `<th>${header}</th>`;
      });
      html += '</tr>';

      for (let i = 1; i < this.csvRows.length; i++) {
        html += '<tr>';
        for (let j = 0; j < this.csvRows[i].length; j++) {
          let cellData = this.csvRows[i][j];
          if (j === 0) {
            try {
              const response = await axios(cellData, {
                method: 'GET',
                mode: 'no-cors',
              });
              cellData = `<img src="${response.request.responseURL}" alt="Image" style="width: 100px;">`;
              console.log(response);
            } catch (error) {
              console.log('');
            }
          }
          html += `<td>${cellData}</td>`;
        }
        html += '</tr>';
        this.progress = Math.round((i / (this.csvRows.length - 1)) * 100);
      }

      html += '</table>';
      this.tableHtml = html;
      // Reset progress after completion
      //if (this.progress === 100) setTimeout(() => this.progress = 0, 2000);
    },
    saveHtml() {
      const blob = new Blob([this.tableHtml], { type: 'text/html' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'table.html';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },
  },
};
</script>
