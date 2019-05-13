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
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="提现账号">
          <a-input
            placeholder="请输入提现账号"
            v-decorator="['cashAccount', validatorRules.cashAccount ]"
          />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="提现金额">
          <a-input placeholder="请输入提现金额" v-decorator="['cashMoney', validatorRules.cashMoney ]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="账号名称">
          <a-input placeholder="请输入账号名称" v-decorator="['cashName', validatorRules.cashName ]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="提现状态">
          <a-input placeholder="请输入提现状态" v-decorator="['cashStatus', validatorRules.cashStatus ]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="提现时间">
          <a-input placeholder="请输入提现时间" v-decorator="['cashTime', validatorRules.cashTime ]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="用户id">
          <a-input placeholder="请输入用户id" v-decorator="['vipId', validatorRules.vipId ]"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import { httpAction } from '@/api/manage'
import pick from 'lodash.pick'

export default {
  name: 'CashModal',
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
        cashAccount: { rules: [{ required: true, message: '请输入提现账号!' }] },
        cashMoney: { rules: [{ required: true, message: '请输入提现金额!' }] },
        cashName: { rules: [{ required: true, message: '请输入账号名称!' }] },
        cashStatus: { rules: [{ required: true, message: '请输入提现状态!' }] },
        cashTime: { rules: [{ required: true, message: '请输入提现时间!' }] },
        vipId: { rules: [{ required: true, message: '请输入用户id!' }] }
      },
      url: {
        add: '/cash/cash/add',
        edit: '/cash/cash/edit'
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
          pick(this.model, 'cashAccount', 'cashMoney', 'cashName', 'cashStatus', 'cashTime', 'vipId')
        )
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