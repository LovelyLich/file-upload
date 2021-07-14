<template>
  <div id="oss">
    <div>
      <div>OSS 上传测试</div>
      <input type="file" @change="handleFileChange" />
      <el-button @click="handleUpload">上传</el-button>
      <el-button @click="handleConvert">转换</el-button>
    </div>
  </div>
</template>

<script>
export default {
  name: "Oss",
  data: () => ({
    container: {
      file: null
    },
    data: []
  }),
  methods: {
    handleFileChange(e) {
      const [file] = e.target.files;
      if (!file) return;
      Object.assign(this.$data, this.$options.data());
      this.container.file = file;
      console.log("container file=" + this.container.file);
    },
    request({ url, method = "get", headers = {} }) {
      return new Promise(function(resolve) {
        const xhr = new XMLHttpRequest();
        xhr.open(method, url);
        Object.keys(headers).forEach(key =>
          xhr.setRequestHeader(key, headers[key])
        );
        xhr.onreadystatechange = function() {
          if (xhr.status === 200 && xhr.readyState === 4) {
            console.log("resptext = " + xhr.responseText);
            resolve(xhr.responseText);
          }
        };
        xhr.send(null);
      });
    },
    async GetStsToken() {
      var cred = {};
      await this.request({
        url: "http://47.108.199.150:3000/get_sts_token"
      }).then(function(resp) {
        console.log("resp = " + resp);
        cred = JSON.parse(resp);
      });
      console.log(cred);
      return cred;
    },
    async handleConvert() {
      await this.request({ url: "http://47.108.199.150:3000/convert" }).then(
        function(resp) {
          console.log("resp = " + resp);
        }
      );
      alert("convert request sended!");
    },
    async handleUpload() {
      let resp = await this.GetStsToken();
      let OSS = require("ali-oss");

      console.log("=============>got credential =" + resp);
      let client = new OSS({
        // yourRegion填写Bucket所在地域。以华东1（杭州）为例，Region填写为oss-cn-hangzhou。
        region: "oss-cn-hangzhou",
        // 从STS服务获取的临时访问凭证。临时访问凭证包括临时访问密钥（AccessKeyId和AccessKeySecret）和安全令牌（SecurityToken）。
        accessKeyId: resp.Credentials.AccessKeyId,
        accessKeySecret: resp.Credentials.AccessKeySecret,
        stsToken: resp.Credentials.SecurityToken,
        // 填写Bucket名称。
        bucket: "aim"
      });
      async function put(file) {
        try {
          // object表示上传到OSS的文件名称。
          // file表示浏览器中需要上传的文件，支持HTML5 file和Blob类型。
          let r1 = await client.put("GisUploads/uploads/testfile", file);
          console.log("put success: %j", r1);
        } catch (e) {
          console.error("error: %j", e);
        }
      }
      console.log("this=" + this);
      put(this.container.file);
    }
  }
};
</script>

<style scoped></style>
