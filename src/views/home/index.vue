<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="addDialog.visible=true">上传文件</el-button>
    </div>

    <el-table
      v-loading="listLoading"
      :data="content"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row>
      <el-table-column
        type="index"
        width="50">
      </el-table-column>
      <el-table-column label="名称">
        <template slot-scope="scope">
          {{scope.row}}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="260" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button size="mini" type='primary' @click="check(scope.row)"> 校验 </el-button>
          <el-button size="mini" @click="download(scope.row)"> 下载 </el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog
      :visible="addDialog.visible"
      title="添加文件"
      v-loading="false"
      :before-close="closeAddDialog">
      <el-upload
        class="upload-demo"
        ref="upload"
        action="/api/files"
        :file-list="fileList"
        :on-success="onAddFileSuccess"
        :auto-upload="false">
        <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
      </el-upload>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" v-loading="addDialog.adding" @click="submitUpload">添加</el-button>
        <el-button @click="addDialog.visible = false">取消</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import merge from 'webpack-merge'
import { mapGetters } from 'vuex'

export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  components: {
  },
  computed: {
    ...mapGetters([
      'user'
    ])
  },
  data() {
    return {
      listLoading: true,
      content: null,
      addDialog: {
        visible: false,
        adding: false,
      },
      fileList: []
    }
  },
  created() {
    this.reloadPage()
  },
  methods: {
    getList: async function(info) {
      this.listLoading = true
      await this.ajax.get('/files').then(res => {
        this.content = res.info
      })
      this.listLoading = false
    },
    reloadPage: function() {
      this.getList()
    },
    closeAddDialog() {
      this.addDialog.visible = false
    },
    async check(filename) {
      let result
      await this.ajax.get('/files/check/' + filename).then(res => {
        if (res.info) {
          this.$message({
            message: "校验成功，文件准确。",
            offset: 90,
            type: "success"
          })
          result = true
        } else {
          this.$message({
            message: "校验失败，文件与上传时的版本不相同。",
            offset: 90,
            type: "error"
          })
          result = false
        }
      })
      return result
    },
    async download(filename) {
      const correct = await this.check(filename)
      if (!correct) {
        return
      }
      window.location.href = '/api/files/' + filename
    },
    async submitUpload() {
      this.addDialog.adding = true
      await this.$refs.upload.submit();
    },
    async onAddFileSuccess() {
      this.closeAddDialog()
      this.reloadPage()
      this.fileList = []
      this.addDialog.adding = false

    }
  }
}
</script>
