<template>
  <a-drawer
    :title="title"
    :maskClosable="true"
    :width="800"
    :visible="visible"
    placement="right"
    :closable="true"
    @close="handleCancel"

  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="最低提现金额">
          <a-input placeholder="请输入最低提现金额" v-decorator="['lowMoney', {}]"/>
        </a-form-item>
         <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="提现说明">
          <a-input placeholder="请输入提现说明" v-decorator="['repaymentNotice', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="充值说明">
          <editor v-model="buyDesc" :init="init" :disabled="disabled" @onClick="onClick"></editor>
        </a-form-item>
      
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="新手教程">
          <editor v-model="newCourse" :init="init" :disabled="disabled" @onClick="onClick"></editor>
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
  name: 'FanliModal',
  data() {
    return {
      title: '操作',
      visible: false,
      uploadLoading: false,
      disableSubmit: false,
      confirmLoading: false,
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
      validatorRules: {},
      url: {
        add: '/fanli/fanli/add',
        edit: '/fanli/fanli/edit'
      },
      //初始化配置
      init: {
        language_url: '/tinymce/langs/zh_CN.js',
        language: 'zh_CN',
        skin_url: '/tinymce/skins/lightgray',
        height: 300,
        plugins: this.plugins,
        toolbar: this.toolbar,
        branding: false,
        menubar: false,
        images_upload_handler: (blobInfo, success) => {
          const img = 'data:image/jpeg;base64,' + blobInfo.base64()
          success(img)
        }
      },
      buyDesc: this.value,
      newCourse: this.value
    }
  },
   mounted() {
    tinymce.init({})
  },
  created() {
    const token = Vue.ls.get(ACCESS_TOKEN)
    this.headers = { 'X-Access-Token': token }
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
        this.buyDesc = ''
        this.newCourse = ''
      } else {
        this.buyDesc = record.buyDesc
        this.newCourse = record.newCourse
      }
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'lowMoney', 'repaymentNotice'))
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
      this.disableSubmit = false
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

          this.model.buyDesc = this.buyDesc
          this.model.newCourse = this.newCourse
          
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