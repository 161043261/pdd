<script>
import axios from "axios";

export default {
  name: "main-content",
  data() {
    return {
      url: "http://127.0.0.1:2023/",
      urlL: "",
      urlR: "",
      arrL: [],
      arrR: [],
      infoArr: [],
    };
  },
  created: function () {
    document.title = "Plant Disease Detection";
  },
  methods: {
    preUpload() {
      this.$refs.upload.click();
    },

    upload(e) {
      const file = e.target.files[0];
      const param = new FormData();
      param.append("file", file, file.name);
      const arr = file.name.toLowerCase().split(".");
      const extName = arr[arr.length - 1];
      if (extName === "jpg" || extName === "jpeg" || extName === "png") {
        axios
          .post(this.url + "file", param, {
            headers: { "Content-Type": "multipart/form-data" },
          })
          .then((response) => {
            this.urlL = response.data.imageUrl;
            this.urlR = response.data.imageOutUrl;
            this.arrL.push(this.urlL);
            this.arrR.push(this.urlR);
            const classes = Object.keys(response.data.targetInfo);
            this.infoArr = [];
            for (let i = 0; i < classes.length; i++) {
              response.data.targetInfo[classes[i]][2] = classes[i];
              this.infoArr.push(response.data.targetInfo[classes[i]]);
            }
            this.$notify({
              title: "Success",
              message: "Click to preview",
              duration: 3000,
            });
          });
      } else if (extName === "avi" || extName === "mov" || extName === "mp4") {
        axios
          .post(this.url + "file", param, {
            headers: { "Content-Type": "multipart/form-data" },
          })
          .then((response) => {
            console.log(response.data);
            this.download(response.data.videoPath, {
              responseType: "blob",
            });
          });
      } else {
        this.$notify({
          title: "Failure",
          message: "Unsupported file extensions",
          duration: 3000,
        });
      }
    },

    download(filePath, config) {
      const arr = filePath.split("/");
      const filename = arr[arr.length - 1];
      let title = "Success";
      axios
        .get(this.url + filePath, config)
        .then((response) => {
          const blob = new Blob([response.data]);
          const eLink = document.createElement("a");
          eLink.download = filename;
          eLink.style.display = "none";
          eLink.href = URL.createObjectURL(blob);
          document.body.appendChild(eLink);
          eLink.click();
          URL.revokeObjectURL(eLink.href);
          document.body.removeChild(eLink);
        })
        .catch((err) => {
          console.log(err === null);
          title = "Failure";
        })
        .finally(() => {
          this.$notify({ title, message: filename, duration: 3000 });
        });
    },
  },
};
</script>

<template>
  <div id="content">
    <el-card shadow="always">
      <div slot="header">
        Plant Disease Detection
        <el-button type="success" v-on:click="preUpload"
          >upload image / video
          <input
            ref="upload"
            style="display: none"
            type="file"
            @change="upload"
          />
        </el-button>
      </div>
      <el-card shadow="always">
        <el-row type="flex" justify="center">
          <el-image
            :src="urlL"
            class="preview-container"
            :preview-src-list="arrL"
          ></el-image>
          <el-image
            :src="urlR"
            class="preview-container"
            :preview-src-list="arrR"
          ></el-image>
        </el-row>
      </el-card>
      <el-table
        :data="infoArr"
        border
        style="width: 100%"
        :header-cell-style="{ 'text-align': 'center' }"
        :cell-style="{ 'text-align': 'center' }"
      >
        <el-table-column label="class">
          <template #default="scope">
            <span>{{ scope.row[2] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="confidence">
          <template #default="scope">
            <span>{{ scope.row[1] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="bbox">
          <template #default="scope">
            <span>{{ scope.row[0] }}</span>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<style scoped>
.preview-container {
  width: 400px;
  height: 400px;
  border: 3px solid lightgreen;
  border-radius: 30px;
  margin: 20px;
}
</style>
