<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="手机号">
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
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!--图片预览-->
    <template>
      <div>
        <a-modal title="图片预览" v-model="visible" @ok="handleOk">
          <img :src="imageUrl" style="width: 100%;height: 100%;">
        </a-modal>
      </div>
    </template>

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
    </div> -->

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
        <template slot="imageSlots" slot-scope="text, record">
          <div class="anty-img-wrap">
            <img :src="getAvatarView(record.avater)" @click="showImg(record.avater)">
          </div>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a @click="handleEdit(record)">充值</a>
          <a-divider type="vertical"/>
          <a @click="handleEdit(record)">提现</a>
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
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a></a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <vipUser-modal ref="modalForm" @ok="modalFormOk"></vipUser-modal>
  </a-card>
</template>

<script>
import VipUserModal from './modules/VipUserModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'

export default {
  name: 'VipUserList',
  mixins: [JeecgListMixin],
  components: {
    VipUserModal
  },
  data() {
    return {
      visible: false,
      imageUrl: '',
      description: '用户管理管理页面',
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
          title: '头像',
          align: 'center',
          dataIndex: 'avater',
          scopedSlots: { customRender: 'imageSlots' }
        },
        {
          title: '钱包余额',
          align: 'center',
          dataIndex: 'bucketmoney'
        },
        {
          title: '充值总额',
          align: 'center',
          dataIndex: 'buymoney'
        },
        {
          title: '昵称',
          align: 'center',
          dataIndex: 'nickname'
        },
        {
          title: '手机号',
          align: 'center',
          dataIndex: 'phone'
        },
        {
          title: '资产总额',
          align: 'center',
          dataIndex: 'total'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        imgerver: window._CONFIG['domianURL'] + '/sys/common/view',
        list: '/vipuser/vipUser/list',
        delete: '/vipuser/vipUser/delete',
        deleteBatch: '/vipuser/vipUser/deleteBatch',
        exportXlsUrl: 'vipuser/vipUser/exportXls',
        importExcelUrl: 'vipuser/vipUser/importExcel'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
     getAvatarView(image) {
      return this.url.imgerver + '/' + image
    },
    showImg(image) {
      this.visible = true
      this.imageUrl = this.url.imgerver + '/' + image
    },
    handleOk(e) {
      console.log(e);
      this.visible = false
    },
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

.anty-img-wrap {
  height: 50px;
  position: relative;
}

.anty-img-wrap > img {
  max-height: 100%;
}
</style>