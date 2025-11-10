<script>
import axios from "axios";

export default {
  name: "main-content",
  data() {
    return {
      url: "http://127.0.0.1:2023/",
      leftUrl: "",
      rightUrl: "",
      leftArr: [],
      rightArr: [],
      infoArr: [],
    };
  },
  created() {
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
            this.leftUrl = response.data.imageUrl;
            this.rightUrl = response.data.imageOutUrl;
            this.leftArr.push(this.leftUrl);
            this.rightArr.push(this.rightUrl);
            const keys = Object.keys(response.data.targetInfo);
            this.infoArr = [];
            for (let i = 0; i < keys.length; i++) {
              response.data.targetInfo[keys[i]][2] = keys[i];
              this.infoArr.push(response.data.targetInfo[keys[i]]);
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
          const anchor = document.createElement("a");
          anchor.download = filename;
          anchor.style.display = "none";
          anchor.href = URL.createObjectURL(blob);
          document.body.appendChild(anchor);
          anchor.click();
          URL.revokeObjectURL(anchor.href);
          document.body.removeChild(anchor);
        })
        .catch(() => {
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
        <el-button type="success" v-on:click="preUpload">
          <label for="upload">Upload Image/Video</label>
          <input
            id="upload"
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
            :src="leftUrl"
            class="preview-container"
            :preview-src-list="leftArr"
          />
          <el-image
            :src="rightUrl"
            class="preview-container"
            :preview-src-list="rightArr"
          />
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
  width: 30rem;
  height: 30rem;
  border: 0.1rem solid lightgreen;
  border-radius: 1rem;
  margin: 1rem;
}
</style>
