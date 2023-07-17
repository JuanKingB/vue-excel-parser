<template>
  <div>
    <label class="file-select">
      <div class="select-button">
        <span v-if="value">Selected File: {{ value.name }}</span>
        <span v-else>Select File</span>
      </div>
      <input type="file" @change="handleFileUpload" accept=".xlsx, .xls" />
    </label>
  </div>
  <div class="container" v-if="this.json_array.length > 0">
    <div class="group" v-for="(group, index) in this.json_array" :key="index">
      <div class="row">
        <div class="left">
          <div class="title">
            {{
              group.title
            }}
          </div>
        </div>
        <ul class="right">
          <li class="item" v-for="(data, index) in group.datas" :key="index">
            {{
              data
            }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  data() {
    return {
      json_array: [],
    };
  },
  methods: {
    handleFileUpload(event) {
      if (this.json_array.length)
        this.json_array = [];

      const file = event.target.files[0];
      const reader = new FileReader();
      if (!file) return
      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const worksheet = workbook.Sheets[workbook.SheetNames[0]];
        let jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 })
          .filter((data) => data.length > 0); // filter out empty data
        const length = jsonData.length; // length of json_array
        
        // remove the first row from jsonData as it just contains information about the spreadsheet for the user
        jsonData = jsonData.slice(1, length)

        // push data to group array.
        for (var groupIdx2 = 1; groupIdx2 < length; groupIdx2++) { // 
          var title = jsonData[0][groupIdx2] // first data is title
          var datas = [];   // array to store data
          // store data into array
          for (var dataIdx = 1; dataIdx < jsonData.length; dataIdx++) {
            var rowdata = jsonData[dataIdx]
            if (rowdata[groupIdx2])
              datas.push(rowdata[groupIdx2])
          }
          // push data(title & datas) into json_array
          if (title) {
            this.json_array.push({
              title: title,
              datas: datas
            })
          }
        }
      };

      reader.readAsArrayBuffer(file);
    },
  },
};
</script>

<style scoped>
.file-select>.select-button {
  padding: 1rem;

  color: white;
  background-color: #2EA169;

  border-radius: .3rem;

  text-align: center;
  font-weight: bold;
}

.file-select>input[type="file"] {
  display: none;
}

.container {}

.group {
  margin: 25px;
}

.row {
  display: flex;
}


.left {
  padding: 25px;
  width: 40%;
  float: left;
}

.right {
  position: relative;
  border: 1px solid #ccc;
  -webkit-border-radius: 10px;
  -moz-border-radius: 10px;
  -border-radius: 10px;
  border-radius: 10px;
  width: 60%;
  left: 0px;
  float: right;
}

.title {}

.item {
  padding: 25px;
  border-bottom: 1px solid #ccc;
}

ul {
  list-style-type: none;
  padding: 0;
  border: 1px solid #ddd;
}

ul li {
  padding: 8px 16px;
  border-bottom: 1px solid #ddd;
}

ul li:last-child {
  border-bottom: none
}
</style>
