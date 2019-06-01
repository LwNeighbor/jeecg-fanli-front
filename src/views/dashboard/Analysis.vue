<template>
  <div class="page-header-index-wide">
    <a-row :gutter="24">
      <a-col :sm="24" :md="12" :xl="8" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="累计注册量" total=""><h1>{{vipCount}}人</h1>
          <a-tooltip title="平台总共注册的会员数量" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <template slot="footer">
            <trend flag="" style="margin-right: 16px;">
              <span slot="term">累计注册量</span>
              <h3>{{vipCount}} 人</h3>
            </trend>
          </template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="8" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="累计充值金额" total=""><h1>￥ {{ rechargeMoney }}</h1>
          <a-tooltip title="会员总共充值的金额" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <template slot="footer">
            <trend flag="" style="margin-right: 16px;">
              <span slot="term">充值总额</span>
              <h3>￥ {{rechargeMoney}}</h3>
            </trend>
            <trend flag="">
              <span slot="term">充值笔数</span>
              <h3>{{rechargeCount}}</h3>
            </trend>
          </template>
          
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="8" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="累计提现金额" total=""><h1>￥ {{cashMoney}}</h1>
          <a-tooltip title="会员总共提现的金额" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
         <template slot="footer">
            <trend flag="" style="margin-right: 16px;">
              <span slot="term">提现总额</span>
              <h3>￥ {{cashMoney}}</h3>
            </trend>
            <trend flag="">
              <span slot="term">提现笔数</span>
              <h3>{{cashCount}}</h3>
            </trend>
          </template>
        </chart-card>
      </a-col>
    </a-row>
    <a-row :gutter="24">
      <a-col :sm="24" :md="12" :xl="8" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="今日注册量" total=""><h1>{{dayUser}}人</h1>
          <a-tooltip title="平台今日总共注册的会员数量" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <template slot="footer">
            <trend flag="" style="margin-right: 16px;">
              <span slot="term">今日注册量</span>
              <h3>{{dayUser}} 人</h3>
            </trend>
          </template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="8" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="今日充值金额"><h1>￥ {{rechargeMoneyDay}}</h1>
          <a-tooltip title="会员今日总共充值的金额" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <template slot="footer">
            <trend flag="" style="margin-right: 16px;">
              <span slot="term">充值总额</span>
              <h3>￥ {{rechargeMoneyDay}}</h3>
            </trend>
            <trend flag="">
              <span slot="term">充值笔数</span>
              <h3>{{rechargeCountDay}}</h3>
            </trend>
          </template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="8" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="今日提现总额" total=""><h1>￥ {{cashMoneyDay}}</h1>
          <a-tooltip title="会员今日总共提现的金额" slot="action">
            <a-icon type="info-circle-o" />
          </a-tooltip>
          <template slot="footer">
            <trend flag="" style="margin-right: 16px;">
              <span slot="term">提现总额</span>
              <h3>￥ {{cashMoneyDay}}</h3>
            </trend>
            <trend flag="">
              <span slot="term">提现笔数</span>
              <h3>{{cashCountDay}}</h3>
            </trend>
          </template>
        </chart-card>
      </a-col>
    </a-row>
  </div>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import ChartCard from '@/components/ChartCard'
  import ACol from "ant-design-vue/es/grid/Col"
  import ATooltip from "ant-design-vue/es/tooltip/Tooltip"
  import MiniArea from '@/components/chart/MiniArea'
  import MiniBar from '@/components/chart/MiniBar'
  import MiniProgress from '@/components/chart/MiniProgress'
  import RankList from '@/components/chart/RankList'
  import Bar from '@/components/chart/Bar'
  import Trend from '@/components/Trend'

  export default {
    name: "Analysis",
    components: {
      ATooltip,
      ACol,
      ChartCard,
      MiniArea,
      MiniBar,
      MiniProgress,
      RankList,
      Bar,
      Trend
    },
    data() {
      return {
        loading: true,
        center: null,
        loginfo:{},
        vipCount: '',
        cashCount: '',
        cashMoney: '',
        rechargeCount: '',
        rechargeMoney: '',
        dayUser: '',
        cashCountDay: '',
        cashMoneyDay: '',
        rechargeCountDay: '',
        rechargeMoneyDay: '',
        url: {
          list: '/home/analysis/'
        }
      }
    },
    created() {
      setTimeout(() => {
        this.loading = !this.loading
      }, 1000)
      this.initLogInfo();
    },
    methods: {
      initLogInfo () {
        let httpurl = this.url.list
        let method = 'get'
       
        httpAction(httpurl, '', method)
          .then(res => {
            if (res.success) {
             // this.$message.success(res.message)
              this.vipCount = res.result.vipCount;
              this.cashCount = res.result.cashCount;
              this.cashMoney = res.result.cashMoney;
              this.rechargeCount = res.result.rechargeCount;
              this.rechargeMoney = res.result.rechargeMoney;
              this.dayUser = res.result.dayUser;
              this.cashCountDay = res.result.cashCountDay;
              this.cashMoneyDay = res.result.cashMoneyDay;
              this.rechargeCountDay = res.result.rechargeCountDay;
               this.rechargeMoneyDay = res.result.rechargeMoneyDay;

            } else {
              //this.$message.warning(res.message)
            }
          })
      },
    }
  }
</script>

<style lang="scss" scoped>
  .extra-wrapper {
    line-height: 55px;
    padding-right: 24px;

    .extra-item {
      display: inline-block;
      margin-right: 24px;

      a {
        margin-left: 24px;
      }
    }
  }

  /* 首页访问量统计 */
  .head-info {
    position: relative;
    text-align: left;
    padding: 0 32px 0 0;
    min-width: 125px;

    &.center {
      text-align: center;
      padding: 0 32px;
    }

    span {
      color: rgba(0, 0, 0, .45);
      display: inline-block;
      font-size: .95rem;
      line-height: 42px;
      margin-bottom: 4px;
    }
    p {
      line-height: 42px;
      margin: 0;
      a {
        font-weight: 600;
        font-size: 1rem;
      }
    }
  }
</style>