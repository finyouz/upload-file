<script setup>
import { UploadFilled } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus';
import * as PDF from 'pdfjs-dist'
import { ref } from 'vue';
PDF.GlobalWorkerOptions.workerSrc = '../public/pdf.worker.js' 
const pageNum = ref(null)
const password = ref('')
const dialogTableVisible = ref(false)
const uintArray = ref()

//文件上传之前的
const handleBefore = (rawFile)=>{
   
  parsePDF(rawFile) 
}


const parsePDF = (file)=>{
  const reader = new FileReader()
  reader.onload = function(e) {  
        const typedarray = new Uint8Array(e.target.result);  
        parsePdfBytes(typedarray);  
  };  
  reader.readAsArrayBuffer(file); 
  
}

const parsePdfBytes = async(uint8Array)=>{
  
  
  try {
    const loadingTask = PDF.getDocument({
      data:uint8Array,
      password:password.value
    });
    
    const pdf = await loadingTask.promise; 

    if(pdf){
      pageNum.value = pdf._pdfInfo.numPages
      ElMessage.success("解析成功")
    }
    
    
  } catch (error) {
      if(error.code === 1){
        ElMessage.error('该文件加密,需要输入密码')
        dialogTableVisible.value = true
        uintArray.value = uint8Array
      }else if(error.code === 2){
        ElMessage.error('密码错误')
        uintArray.value = uint8Array
      }
    
  }
  
}

const handleConfirm = ()=>{
  dialogTableVisible.value = false

  parsePdfBytes(uintArray.value)
}

const handleCancel = ()=>{
  dialogTableVisible.value = false
  password.value = ''
}
</script>

<template>
  <div class="container">
    <header class="title">
      PDF 文件权限校验与信息展示
    </header>
    <div class="content">
      <div class="upload-file">
        <el-upload class="upload-demo" drag action="https://run.mocky.io/v3/9d059bf9-4660-45f2-925d-ce80ad6c4d15"
          :before-upload="handleBefore" accept=".pdf">
          <el-icon class="el-icon--upload"><upload-filled /></el-icon>
          <div class="el-upload__text">
           请选择你需要上传的文件 
          </div>
          <template #tip>
            <div class="el-upload__tip">
              只支持pdf文件
            </div>
          </template>
        </el-upload>
      </div>
      <div class="information" >
          <div v-if="pageNum===null?'':pageNum">
              <div class="file-content">
              当前文件页数:{{ pageNum }}
            </div>
            <div class="password">
              当前文件的密码:{{ password }}
            </div>
          </div>
      </div>
    </div>
  </div>

  <el-dialog v-model="dialogTableVisible" width="300px">
    <template #header>
      <div>请输入你的密码</div>
    </template>

    <el-input v-model="password" type="password" placeholder="请输入你的密码"></el-input>
   
      <template #footer>
      <div class="dialog-footer">
        <el-button @click="handleCancel">取消</el-button>
        <el-button type="primary" @click="handleConfirm">
          确定
        </el-button>
      </div>
    </template>
 
  </el-dialog>
</template>

<style scoped>
.container {
  width: 1200px;
  height: 100vh;
  margin: 0 auto;

  .title {
    font-size: 30px;
    font-weight: 600;
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .content {
    height: calc(100vh - 50px);
    display: flex;

    .upload-file {
      flex: 1;
      display: flex;
      align-items: center;

      .upload-demo{
        width: 100%;

        .el-upload__tip{
          text-align: center;
        }
      }
    }

    .information {
      flex: 1;
      display: flex;
      flex-direction: column;
      justify-content: center;
      margin-left: 40px;
    }
  }

}
</style>
