<template>
  <div class="app-container">
    <el-card shadow="always">
      <div slot="header" class="clearfix">
        <span class="card_header">基本参数</span>
      </div>
        <span>订单： </span>
        <upload-excel-component :on-success="handleSuccess" :before-upload="beforeUpload" :content="'选择订单'" />
          
        <span style="margin-left:30px">批次容量：</span>
        <el-input v-model="batchNum" style="width:120px" placeholder="请输入容量"></el-input>
        <!-- <div style="margin-top: 15px;"> -->
        <span style="margin-left:30px">分批分配算法：</span>
        <el-select
          filterable
          v-model="currentBatchAlgorith"
          placeholder="分批算法"
          style="margin-right:15px;width:20%;">
          <el-option
            v-for="(item, idx) in batchAlgorithmList"
            :key="idx"
            :label="`${item.label}`"
            :value="item.value"></el-option>
        </el-select>
        <span>拣选路径算法： </span>
        <el-select
          filterable
          v-model="currentPickAlgorith"
          placeholder="拣选算法"
          style="margin-right:15px;">
          <el-option v-for="(item, idx) in pickAlgorithmList"
            :key="idx"
            :label="item.label"
            :value="item.value"></el-option>
        </el-select>
        <el-button
          @click="getBatchResult"
          type="primary">
          查看分批
        </el-button>
        <!-- </div> -->
      </el-card>
    <el-card style="margin-top:10px" shadow="always" :body-style="{ padding: '10px' }">
      <div slot="header" class="clearfix">
        <span class="card_header">待处理订单</span>
      </div>
      <el-table 
          :data="tableData"
          :border="showBorder" 
          stripe 
          height="350"
          highlight-current-row
          style="width: 100%;"
      >
          <el-table-column sortable v-for="item of tableHeader" :key="item" :prop="item" :label="item" />
      </el-table>
    </el-card>
    <el-card style="margin-top:10px" shadow="always" :body-style="{ padding: '10px' }">
      <div slot="header" class="clearfix">
        <span class="card_header">分批结果</span>
      </div>
      <upload-excel-component v-show="uploadButton" :on-success="handleSuccess2" :before-upload="beforeUpload" :content="'选择分批结果订单'" />
      <el-table
          :data="tableData2"
          border
          stripe
          height="350"
          highlight-current-row
          style="width: 100%;"
      >
          <el-table-column sortable v-for="item of tableHeader2" :key="item" :prop="item" :label="item" />
      </el-table>
    </el-card>
    <el-card style="margin-top:10px" shadow="always">
      <div slot="header" class="clearfix">
        <span class="card_header">分批顺序结果</span>
      </div>
      <p v-for="(item, index) in batchResults" :key="index">
        {{item.name}}:<span v-for="(item, index) in item.path" :key="index">{{index == 0 ? `${item}` : `--${item}`}}</span>
      </p>
    </el-card>
  </div>
</template>

<script>
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
export default {
  components: { UploadExcelComponent },
  data() {
    return {
      batchNum: 40,
      showBorder: false,
      tableData: [],
      tableHeader: [],
      tableData2: [],
      tableHeader2: [],
      uploadButton: true,
      currentBatchAlgorith: '选项1',
      currentPickAlgorith: '选项2',
      batchAlgorithmList: [
        {
          value: '选项1',
          label: '分批算法1'
        }, {
          value: '选项2',
          label: '分批算法2'
        }
      ],
      pickAlgorithmList: [
        {
          value: '选项1',
          label: '拣选算法1'
        }, {
          value: '选项2',
          label: '拣选算法2'
        }
      ],
      //分批结果
      batchResults: {
        piker1: {
          name: '拣货员1',
          path: [
            '批次0228001',
            '批次0228002',
            '批次0228003'
          ]
        },
        piker2: {
          name: '拣货员2',
          path: [
            '批次0328004',
            '批次0328005',
            '批次0328006'
          ]
        }
      }
    }
  },
  created() {
      localStorage.setItem('batchResults', JSON.stringify(this.batchResults))
      // let pickerList = [];
      // Object.keys(this.results).forEach((item) => {
      //   console.log(item);
      //   pickerList.push(item);
      // })
      // let currentPiker = pickerList.length ?  pickerList[0] : ''
      // localStorage.setItem('currentPiker', currentPiker)
      // localStorage.setItem('pickerList', pickerList)
  },
  methods: {
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
    },
    handleSuccess2({ results, header }) {
      this.tableData2 = results
      console.log('分批结果数据数组',results);
      this.tableHeader2 = header
      this.uploadButton = false
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
.el-card__body{
  padding: 10px
}
</style>

