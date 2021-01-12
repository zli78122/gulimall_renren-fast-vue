<template>
  <el-dialog
    title="Cloud Storage Config"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
      <el-form-item size="mini" label="Type">
        <el-radio-group v-model="dataForm.type">
          <el-radio :label="1">Alibaba Cloud OSS</el-radio>
          <el-radio :label="2">AWS S3</el-radio>
          <el-radio :label="3">GCS</el-radio>
        </el-radio-group>
      </el-form-item>
      <template v-if="dataForm.type === 1">
        <el-form-item label="Domain">
          <el-input v-model="dataForm.aliyunDomain" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="Path Prefix">
          <el-input v-model="dataForm.aliyunPrefix" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="EndPoint">
          <el-input v-model="dataForm.aliyunEndPoint" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="AccessKeyId">
          <el-input v-model="dataForm.aliyunAccessKeyId" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="AccessKeySecret">
          <el-input v-model="dataForm.aliyunAccessKeySecret" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="BucketName">
          <el-input v-model="dataForm.aliyunBucketName" placeholder=""></el-input>
        </el-form-item>
      </template>
      <template v-else-if="dataForm.type === 2">
        <!-- <el-form-item size="mini">
          <a href="http://www.renren.io/open/qiniu.html" target="_blank">免费申请(七牛)10GB储存空间</a>
        </el-form-item> -->
        <el-form-item label="Domain">
          <el-input placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="Path Prefix">
          <el-input placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="AccessKey">
          <el-input placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="SecretKey">
          <el-input placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="Space Name">
          <el-input placeholder=""></el-input>
        </el-form-item>
      </template>
      <template v-else-if="dataForm.type === 3">
        <el-form-item label="Domain">
          <el-input v-model="dataForm.qcloudDomain" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="Path prefix">
          <el-input v-model="dataForm.qcloudPrefix" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="AppId">
          <el-input v-model="dataForm.qcloudAppId" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="SecretId">
          <el-input v-model="dataForm.qcloudSecretId" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="SecretKey">
          <el-input v-model="dataForm.qcloudSecretKey" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="BucketName">
          <el-input v-model="dataForm.qcloudBucketName" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="BucketArea">
          <el-input v-model="dataForm.qcloudRegion" placeholder=""></el-input>
        </el-form-item>
      </template>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {},
        dataRule: {}
      }
    },
    methods: {
      init (id) {
        this.visible = true
        this.$http({
          url: this.$http.adornUrl('/sys/oss/config'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.dataForm = data && data.code === 0 ? data.config : []
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl('/sys/oss/saveConfig'),
              method: 'post',
              data: this.$http.adornData(this.dataForm)
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>

