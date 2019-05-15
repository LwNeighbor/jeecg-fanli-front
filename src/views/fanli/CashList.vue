<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="会员手机号">
              <a-input placeholder="请输入手机号" v-model="queryParam.phone"></a-input>
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
              <!-- <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a> -->
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
     <!--  <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button> -->
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
    </div>

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
      >
        <span slot="action" slot-scope="text, record">
          <!-- <a @click="handleEdit(record)">编辑</a> -->
          <a-popconfirm
            title="确认打款?"
            @confirm="() => confirmCash(record.id)"
            v-if="record.cashStatus == '1'"
          >
            <a>打款</a>
          </a-popconfirm>
          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">
              更多
              <a-icon type="down"/>
            </a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <cash-modal ref="modalForm" @ok="modalFormOk"></cash-modal>
  </a-card>
</template>

<script>
import { httpAction } from '@/api/manage'
import CashModal from './modules/CashModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'

export default {
  name: 'CashList',
  mixins: [JeecgListMixin],
  components: {
    CashModal
  },
  data() {
    return {
      description: '提现管理页面',
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '用户手机号',
          align: 'center',
          dataIndex: 'phone'
        },
        {
          title: '提现账号',
          align: 'center',
          dataIndex: 'cashAccount'
        },
        {
          title: '提现金额',
          align: 'center',
          dataIndex: 'cashMoney'
        },
        {
          title: '账号名称',
          align: 'center',
          dataIndex: 'cashName'
        },
        {
          title: '提现状态',
          align: 'center',
          dataIndex: 'cashStatus_dictText'
        },
        {
          title: '提现时间',
          align: 'center',
          dataIndex: 'cashTime'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        cashUrl: '/cash/cash/cash',
        list: '/cash/cash/list',
        delete: '/cash/cash/delete',
        deleteBatch: '/cash/cash/deleteBatch'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    confirmCash(pid) {
      const that = this
      that.confirmLoading = true
      let httpurl = this.url.cashUrl
      let method = 'put'

      let data = {
        id: pid
      }

      let formData = Object.assign(data)
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