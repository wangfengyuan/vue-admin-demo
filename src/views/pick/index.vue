<template>
  <div class="app-container">
    <el-card shadow="always">
      <div slot="header" class="clearfix">
        <span class="card_header">拣货员批次选择</span>
      </div>
        <span style="margin-left:30px">拣货员：</span>
        <el-select
          filterable
          v-model="currentPicker"
          placeholder="拣货员"
          @change="changePiker"
          style="margin-right:15px;width:15%;">
          <el-option
            v-for="(item, idx) in pickerList"
            :key="idx"
            :label="`${item.label}`"
            :value="item.value"></el-option>
        </el-select>
        <span>拣选路径算法： </span>
        <el-select
          filterable
          v-model="currentBatchInfo"
          placeholder="批次信息"
          style="margin-right:15px;width:30%;">
          <el-option v-for="(item, idx) in batchInfoList"
            :key="idx"
            :label="item.label"
            :value="item.value"></el-option>
        </el-select>
        <el-button
          @click="getBatchResult"
          type="primary">
          查看路径
        </el-button>
        <!-- </div> -->
      </el-card>
    <el-card style="margin-top:10px" shadow="always" :body-style="{ padding: '10px' }">
      <div slot="header" class="clearfix">
        <span class="card_header">路径图</span>
      </div>
      <demo></demo>
      <!-- <el-table 
          :data="tableData"
          :border="showBorder"
          stripe 
          height="350"
          highlight-current-row
          style="width: 100%;"
      >
          <el-table-column sortable v-for="item of tableHeader" :key="item" :prop="item" :label="item" />
      </el-table> -->
    </el-card>
    <el-card style="margin-top:10px" shadow="always" :body-style="{ padding: '10px' }">
      <div slot="header" class="clearfix">
        <span class="card_header">拣选信息</span>
      </div>
      <div style="display:flex; flex-direction:row">
        <div style="flex:1">
          <upload-excel-component v-show="uploadButton1" :on-success="handleSuccess" :before-upload="beforeUpload" :content="'选择分批结果订单'" />
          <div style="display:flex; flex-direction:row;">
              <div class="task_box">拣选任务</div>
              <div style="flex:1">
                  <el-table
                      :data="tableData"
                      border
                      stripe
                      height="250"
                      highlight-current-row
                      style="width: 100%;"
                  >
                      <el-table-column v-for="item of tableHeader" :key="item" :prop="item" :label="item" />
                  </el-table>
              </div>
          </div> 
          
        </div>
        <div style="flex:1">
          <upload-excel-component v-show="uploadButton2" :on-success="handleSuccess2" :before-upload="beforeUpload" :content="'选择分批结果订单'" />
          <div style="display:flex; flex-direction:row;">
              <div class="task_box">库存信息</div>
              <div style="flex:1">
                  <el-table
                      :data="tableData2"
                      border
                      stripe
                      height="250"
                      highlight-current-row
                      style="width: 100%;"
                  >
                      <el-table-column v-for="item of tableHeader2" :key="item" :prop="item" :label="item" />
                  </el-table>
              </div>
          </div>
        </div>
      </div>
      
    </el-card>
  </div>
</template>

<script>
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
import demo from './demo'
export default {
  components: { UploadExcelComponent, demo },
  data() {
    return {
      batchNum: 40,
      showBorder: false,
      tableData: [],
      tableHeader: [],
      tableData2: [],
      tableHeader2: [],
      uploadButton1: true,
      uploadButton2: true,
      currentPicker: '选项1',
      currentBatchInfo: '选项2',
      pickerList: [],
      batchInfoList: [],
      //分批结果
      batchResults: {}
    }
  },
  computed: {
    getCurrentBatchInfo() {
      this.currentBatchInfo = this.currentPicker
    }
  },
  created() {
      this.batchResults = JSON.parse(localStorage.getItem('batchResults'))
      // console.log(this.batchResults);
      let self = this
      Object.keys(this.batchResults).forEach((item, index) => {
        console.log(item);
        let name = self.batchResults[item].name;
        let path = self.batchResults[item].path;
        self.pickerList.push({
          value: item,
          label: name
        });
      })
      this.currentPicker = this.pickerList.length ?  this.pickerList[0].value : ''
      this.batchResults[this.currentPicker].path.forEach((item, index) => {
        self.batchInfoList.push({
          value: index,
          label: item
        });
      });
      this.currentBatchInfo = this.batchInfoList.length ?  this.batchInfoList[0].value : ''
  },
  methods: {
    changePiker(e) {
      let obj = this.batchResults[e]
      // console.log('obj', obj);
      this.batchInfoList = []
      obj.path.forEach((item, index) => {
        this.batchInfoList.push({
          value: index,
          label: item
        });
      });
      this.currentBatchInfo = this.batchInfoList.length ?  this.batchInfoList[0].value : ''
    },
    uploadFile(params) {
        const _file = params.file;
        const fileReader = new FileReader();
        fileReader.onload = (ev) => {
          try {
            const data = ev.target.result;
            const workbook = XLSX.read(data, {
              type: 'binary'
            });
            for (let sheet in workbook.Sheets) {
             //循环读取每个文件
              const sheetArray = XLSX.utils.sheet_to_json(workbook.Sheets[sheet]);
              console.log("读取文件");
              console.log(sheetArray)
            }
            } catch (e) {
              this.$message.warning('文件类型不正确！');
            }
          };
          fileReader.readAsBinaryString(_file);
    },
    getBatchResult() {

    },
    beforeUpload(file) {
      const isLt1M = file.size / 1024 / 1024 < 1

      if (isLt1M) {
        return true
      }

      this.$message({
        message: 'Please do not upload files larger than 1m in size.',
        type: 'warning'
      })
      return false
    },
    handleSuccess({ results, header }) {
      this.tableData = results
      console.log('待处理订单数据数组',results);
      this.tableHeader = header
      this.uploadButton1 = false
    },
    handleSuccess2({ results, header }) {
      this.tableData2 = results
      console.log('分批结果数据数组',results);
      this.tableHeader2 = header
      this.uploadButton2 = false
    },
    onSubmit() {
      this.$message('submit!')
    },
    onCancel() {
      this.$message({
        message: 'cancel!',
        type: 'warning'
      })
    }
  }
}
</script>

<style scoped>
.line{
  text-align: center;
}
.card_header {
  color: #409EFF;
  font-size: 18px;
  font-weight: 700
}
.task_box {
  background-color: #64a6ea;
  width: 45px;
  margin: 0px 6px;
  padding: 29px 10px 0 10px;
  font-size: 18px;
  letter-spacing: 1.5em;
  writing-mode: vertical-lr;
  text-align: center;
  color: #fff;
  border-radius: 20px;
  /* border: 2px solid #409EFF; */
}
</style>

