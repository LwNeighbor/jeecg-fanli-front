<template>
  <a-drawer
    :title="title"
    :maskClosable="true"
    :width="drawerWidth"
    placement="right"
    :closable="true"
    @close="handleCancel"
    :visible="visible"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="公告标题">
          <a-input
            placeholder="请输入公告标题"
            :v-model="title"
            v-decorator="['title', validatorRules.title ]"
          />
        </a-form-item>
        <a-form-item label="图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-upload
            listType="picture-card"
            class="avatar-uploader"
            :showUploadList="false"
            :action="uploadAction"
            :data="{'isup':1}"
            :headers="headers"
            :beforeUpload="beforeUpload"
            @change="handleChange"
          >
            <img
              v-if="model.noticeImg"
              :src="getAvatarView()"
              alt="图片"
              style="height:104px;max-width:300px"
            >
            <div v-else>
              <a-icon :type="uploadLoading ? 'loading' : 'plus'"/>
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="公告详情">
          <editor v-model="noticeDesc" :init="init" :disabled="disabled" @onClick="onClick"></editor>
        </a-form-item>
      </a-form>
    </a-spin>
    <div class="drawer-bootom-button" v-show="!disableSubmit">
      <a-popconfirm title="确定放弃编辑？" @confirm="handleCancel" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="confirmLoading">提交</a-button>
    </div>
  </a-drawer>
</template>

<script>
import { httpAction } from '@/api/manage'
import Vue from 'vue'
import pick from 'lodash.pick'
import { ACCESS_TOKEN } from '@/store/mutation-types'
import tinymce from 'tinymce/tinymce'
import Editor from '@tinymce/tinymce-vue'
import 'tinymce/themes/silver/theme'
import 'tinymce/plugins/image'
import 'tinymce/plugins/media'
import 'tinymce/plugins/table'
import 'tinymce/plugins/lists'
import 'tinymce/plugins/contextmenu'
import 'tinymce/plugins/wordcount'
import 'tinymce/plugins/colorpicker'
import 'tinymce/plugins/textcolor'

export default {
  components: {
    Editor
  },
  props: {
    value: {
      type: String,
      required: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plugins: {
      type: [String, Array],
      default: 'lists image media table textcolor wordcount contextmenu'
    },
    toolbar: {
      type: [String, Array],
      default:
        'undo redo |  formatselect | bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | lists image media table | removeformat'
    }
  },
  name: 'NoticeModal',
  data() {
    return {
      title: '操作',
      visible: false,
      drawerWidth: 1200,
      uploadLoading: false,
      disableSubmit: false,
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },

      confirmLoading: false,
      form: this.$form.createForm(this),
      validatorRules: {
        title: { rules: [{ required: true, message: '请输入公告标题!' }] }
      },
      url: {
        add: '/notice/notice/add',
        edit: '/notice/notice/edit',
        imgerver: window._CONFIG['domianURL'] + '/sys/common/view',
        fileUpload: window._CONFIG['domianURL'] + '/sys/common/upload'
      },
      //初始化配置
      init: {
        language_url: '/tinymce/langs/zh_CN.js',
        language: 'zh_CN',
        skin_url: '/tinymce/skins/lightgray',
        height: 800,
        plugins: this.plugins,
        toolbar: this.toolbar,
        branding: false,
        menubar: false,
        images_upload_handler: (blobInfo, success) => {
          const img = 'data:image/jpeg;base64,' + blobInfo.base64()
          success(img)
        }
      },
      noticeDesc: this.value
    }
  },
  mounted() {
    tinymce.init({})
  },
  watch: {
    value(newValue) {
      this.noticeDesc = newValue
    },
    noticeDesc(newValue) {
      this.$emit('input', newValue)
    }
  },
  created() {
    const token = Vue.ls.get(ACCESS_TOKEN)
    this.headers = { 'X-Access-Token': token }
  },
  computed: {
    uploadAction: function() {
      return this.url.fileUpload
    }
  },
  methods: {
    onClick(e) {
      this.$emit('onClick', e, tinymce)
    },
    add() {
      this.edit({})
    },
    edit(record) {
      if (JSON.stringify(record) == '{}') {
        this.noticeDesc = ''
      } else {
        this.noticeDesc = record.noticeDesc
      }
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'title', 'noticeImg'))
        //时间格式化
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
      this.disableSubmit = false
    },
    beforeUpload: function(file) {
      var fileType = file.type
      if (fileType.indexOf('image') < 0) {
        this.$message.warning('请上传图片')
        return false
      }
      var fileSize = file.size
      if (fileSize > 5242880) {
        this.$message.warning('图片不能大于5M')
        return false
      }
      //TODO 验证文件大小
    },
    handleChange(info) {
      if (info.file.status === 'uploading') {
        this.uploadLoading = true
        return
      }
      if (info.file.status === 'done') {
        var response = info.file.response
        this.uploadLoading = false
        if (response.success) {
          this.model.noticeImg = response.message
        } else {
          this.$message.warning(response.message)
        }
      }
    },
    getAvatarView() {
      return this.url.imgerver + '/' + this.model.noticeImg
    },
    handleSubmit() {
      const that = this
      // 触发表单验证
      this.form.validateFields((err, values) => {
        if (!err) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          this.model.noticeDesc = this.noticeDesc
          let formData = Object.assign(this.model, values)
          //时间格式化

          httpAction(httpurl, formData, method)
            .then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            })
            .finally(() => {
              that.confirmLoading = false
              that.close()
            })
        }
      })
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>

<style scoped>
.avatar-uploader > .ant-upload {
  width: 104px;
  height: 104px;
}
.ant-upload-select-picture-card i {
  font-size: 49px;
  color: #999;
}

.ant-upload-select-picture-card .ant-upload-text {
  margin-top: 8px;
  color: #666;
}

.ant-table-tbody .ant-table-row td {
  padding-top: 10px;
  padding-bottom: 10px;
}
.drawer-bootom-button {
  position: absolute;
  bottom: -8px;
  width: 100%;
  border-top: 1px solid #e8e8e8;
  padding: 10px 16px;
  text-align: right;
  left: 0;
  background: #fff;
  border-radius: 0 0 2px 2px;
}
</style>