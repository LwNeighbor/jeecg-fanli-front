<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="用户手机号">
              <a-input placeholder="请输入用户手机号" v-model="queryParam.phone"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button
                type="primary"
                @click="searchReset"
                icon="reload"
                style="margin-left: 8px"
              >重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <!-- <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>-->

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange"
        @expand="handleExpand"
      >
        <span slot="action" slot-scope="text, record">
          <a-popconfirm
            title="确认中断?"
            @confirm="() => upAndDown(record.id)"
            v-if="record.projectStatus == '1' && record.updown === 'N'"
          >
            <a>中断</a>
          </a-popconfirm>
          
            <a v-if="record.projectStatus == '1' && record.updown === 'Y'">已中断</a>
         
        </span>
        <a-table
          slot="expandedRowRender"
          slot-scope=""
          :columns="innerColumns" 
          :dataSource="innerData"
          size="middle"
          bordered
          rowKey="id"
          :pagination="false"
          :loading="loading"
        ></a-table>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <projectRecord-modal ref="modalForm" @ok="modalFormOk"></projectRecord-modal>
  </a-card>
</template>

<script>
import { getAction } from '@/api/manage'
import ProjectRecordModal from './modules/ProjectRecordModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { httpAction } from '@/api/manage'

export default {
  name: 'ProjectRecordList',
  mixins: [JeecgListMixin],
  components: {
    ProjectRecordModal
  },
  data() {
    return {
      buyDate: '',
      description: '理财记录管理页面',
      // 子表表头
      innerColumns: [
        {
          title: '返利说明',
          align: 'center',
          dataIndex: 'repaymentIntro'
        },
        {
          title: '返利金额',
          align: 'center',
          dataIndex: 'repaymentMoney'
        },
        {
          title: '返利时间',
          align: 'center',
          dataIndex: 'repaymentTime'
        },
        {
          title: '返利状态',
          align: 'center',
          key: 'repaymentStatus',
          customRender: function(text) {
            if (text.repaymentStatus == 1) {
              return '返利中'
            } else if (text.repaymentStatus == 2) {
              return '返利成功'
            } else {
              return text
            }
          }
        }
      ],
      innerData: [],
      expandedRowKeys: [],
      id: ' ',  
      // 表头
      columns: [
        {
          title: '项目名称',
          align: 'center',
          dataIndex: 'projectName'
        },
        {
          title: '手机号',
          align: 'center',
          dataIndex: 'phone'
        },
        {
          title: '购买金额',
          align: 'center',
          dataIndex: 'buyMoney'
        },
        {
          title: '购买时间',
          align: 'center',
          dataIndex: 'buyTime'
        },
        {
          title: '每天返利',
          align: 'center',
          dataIndex: 'dayProfit'
        },
        {
          title: '认购结束',
          align: 'center',
          dataIndex: 'endTime'
        },
        {
          title: '当前天数',
          align: 'center',
          dataIndex: 'nowDay'
        },
        {
          title: '投资天数',
          align: 'center',
          dataIndex: 'projectDay'
        },
        {
          title: '产品状态',
          align: 'center',
          dataIndex: 'projectStatus_dictText'
        },
        {
          title: '起息时间',
          align: 'center',
          dataIndex: 'recordStart'
        },
        {
          title: '汇款日期',
          align: 'center',
          dataIndex: 'repaymentTime'
        },
        {
          title: '认购时间',
          align: 'center',
          dataIndex: 'startTime'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/projectRecord/projectRecord/list',
        delete: '/projectRecord/projectRecord/delete',
        deleteBatch: '/projectRecord/projectRecord/deleteBatch',
        customerListByMainId: '/repaymentRecord/repaymentRecord/inlineList',
        breakUpDown: '/projectRecord/projectRecord/breakUpDown'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    },
    currentId() {
      return this.id
    }
  },
  methods: {
    handleExpand(expanded, record) {
      this.expandedRowKeys = []
      this.innerData = []
      if (expanded === true) {
        this.loading = true
        this.expandedRowKeys.push(record.id)
        getAction(this.url.customerListByMainId, { recordId: record.id }).then(res => {
          if (res.success) {
            this.loading = false
            this.innerData = res.result
          }
        })
      }
    },
    upAndDown(pid) {
      const that = this
      that.confirmLoading = true
      let httpurl = this.url.breakUpDown
      let method = 'post'

      let data = {
        id: pid
      }

      let formData = Object.assign(data)
      //时间格式化

      httpAction(httpurl, formData , method)
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
          this.loadData();
        })
    }
  }
}
</script>
<style lang="less" scoped>
/** Button按钮间距 */
.ant-btn {
  margin-left: 3px;
}
.ant-card-body .table-operator {
  margin-bottom: 18px;
}
.ant-table-tbody .ant-table-row td {
  padding-top: 15px;
  padding-bottom: 15px;
}
.anty-row-operator button {
  margin: 0 5px;
}
.ant-btn-danger {
  background-color: #ffffff;
}

.ant-modal-cust-warp {
  height: 100%;
}
.ant-modal-cust-warp .ant-modal-body {
  height: calc(100% - 110px) !important;
  overflow-y: auto;
}
.ant-modal-cust-warp .ant-modal-content {
  height: 90% !important;
  overflow-y: hidden;
}
</style>