<template>
  <div class="container">
    <el-upload
        :auto-upload="false"
        drag
        accept=".sb3"
        :on-change="change"
        :multiple="false"
        :limit="1"
        action="">
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">只能上传sb3文件</div>
    </el-upload>
    <el-button style="margin-top: 10px;" v-show="file" type="primary" @click="pack">打包</el-button>
  </div>
</template>

<script>
import axios from "axios";
import FileSaver from "file-saver";
import { Loading } from 'element-ui';

const getJSZip = async () => (await import(/* webpackChunkName: "jszip" */ 'jszip')).default;
const downloadURL = (filename, url) => {
  const link = document.createElement('a');
  link.download = filename;
  link.href = url;
  document.body.appendChild(link);
  link.click();
  link.remove();
};
const setFileFast = (zip, path, data) => {
  zip.files[path] = data;
};

export default {
  name: 'Home',
  data(){
    return {
      file: null
    }
  },
  methods: {
    async change(file, fileList) {
      this.file = file
    },
    async pack() {
      let loadingInstance = Loading.service({ fullscreen: true });
      const packageName = 'sb3-to-exe';
      const zipUrl = `/sb3-player-win-ia32.zip`
      const response = await axios({
        method: 'get',
        url: zipUrl,
        responseType: 'arraybuffer'  // 类型必须为arraybuffer
      });
      const electronZip = await (await getJSZip()).loadAsync(response.data);
      const zip = new (await getJSZip());

      for (const path of Object.keys(electronZip.files)) {
        const file = electronZip.files[path];
        // 在zip文件中创建一个内部文件夹
        let newPath = `${packageName}/${path}`;
        // 重命名可执行文件
        newPath = newPath.replace('sb3-player.exe', `${packageName}.exe`);
        setFileFast(zip, newPath, file);
      }

      zip.remove(`${packageName}/resources/app/dist/demo.sb3`);
      zip.file(`${packageName}/resources/app/dist/demo.sb3`,this.file.raw);
      zip.generateAsync({type:"blob"})
      .then((content)=> {
        loadingInstance.close();
        FileSaver(content, `${packageName}.zip`);
      });
    }
  }
}
</script>

<style>
 .container {
   width: 100%;
   height: 100%;
   display: flex;
   flex-direction: column;
   justify-content: center;
   align-items: center;
 }
</style>
