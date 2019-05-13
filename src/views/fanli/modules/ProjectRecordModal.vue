<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="购买金额">
          <a-input placeholder="请输入购买金额" v-decorator="['buyMoney', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="购买时间">
          <a-input placeholder="请输入购买时间" v-decorator="['buyTime', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="每天返利">
          <a-input placeholder="请输入每天返利" v-decorator="['dayProfit', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="认购结束">
          <a-input placeholder="请输入认购结束" v-decorator="['endTime', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="当前天数">
          <a-input placeholder="请输入当前天数" v-decorator="['nowDay', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="投资天数">
          <a-input placeholder="请输入投资天数" v-decorator="['projectDay', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目id">
          <a-input placeholder="请输入项目id" v-decorator="['projectId', validatorRules.projectId ]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="项目介绍">
          <a-input placeholder="请输入项目介绍" v-decorator="['projectIntro', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="产品状态">
          <a-input placeholder="请输入产品状态" v-decorator="['projectStatus', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="起息时间">
          <a-input placeholder="请输入起息时间" v-decorator="['recordStart', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="汇款日期">
          <a-input placeholder="请输入汇款日期" v-decorator="['repaymentTime', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="还款方式">
          <a-input placeholder="请输入还款方式" v-decorator="['repaymentType', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="认购时间">
          <a-input placeholder="请输入认购时间" v-decorator="['startTime', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="会员id">
          <a-input placeholder="请输入会员id" v-decorator="['vipId', validatorRules.vipId ]"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import { httpAction } from '@/api/manage'
import pick from 'lodash.pick'

export default {
  name: 'ProjectRecordModal',
  data() {
    return {
      title: '操作',
      visible: false,
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
        projectId: { rules: [{ required: true, message: '请输入项目id!' }] },
        vipId: { rules: [{ required: true, message: '请输入会员id!' }] }
      },
      url: {
        add: '/projectRecord/projectRecord/add',
        edit: '/projectRecord/projectRecord/edit'
      }
    }
  },
  created() {},
  methods: {
    add() {
      this.edit({})
    },
    edit(record) {
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(
          pick(
            this.model,
            'buyMoney',
            'buyTime',
            'dayProfit',
            'endTime',
            'nowDay',
            'projectDay',
            'projectId',
            'projectIntro',
            'projectStatus',
            'recordStart',
            'repaymentTime',
            'repaymentType',
            'startTime',
            'vipId'
          )
        )
        //时间格式化
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
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
          let formData = Object.assign(this.model, values)
          //时间格式化

          console.log(formData)
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
</style>