<template>
  <div class="flex-container">
    <div id="main">
    <!--<Panel :padding="10">-->
      <!--<div slot="title">ACM Ranklist</div>-->
      <!--<div class="echarts">-->
        <!--<ECharts :options="options" ref="chart" auto-resize></ECharts>-->
      <!--</div>-->
    <!--</Panel>-->
      <div slot="extra">
        <ul class="filter">
          <li>
            <i-switch size="large" v-model="formFilter.myself" @on-change="handleQueryChange">
              <span slot="open">Mine</span>
              <span slot="close">All</span>
            </i-switch>
          </li>
        </ul>
      </div>
    <Table :data="dataRank" :columns="columns" :loading="loadingTable" size="large"></Table>
    <Pagination :total="total" :page-size.sync="limit" :current.sync="page"
                @on-change="getRankData" show-sizer
                @on-page-size-change="getRankData(1)"></Pagination>
    </div>
  </div>
</template>

<script>
  import api from '@oj/api'
  import Pagination from '@oj/components/Pagination'
  import utils from '@/utils/utils'
  import { RULE_TYPE } from '@/utils/constants'

  export default {
    name: 'acm-rank',
    components: {
      Pagination
    },
    data () {
      return {
        formFilter: {
          myself: false
        },
        page: 1,
        limit: 30,
        total: 0,
        routeName: '',
        loadingTable: false,
        dataRank: [],
        columns: [
          {
            align: 'center',
            width: 60,
            render: (h, params) => {
              return h('span', {}, params.index + (this.page - 1) * this.limit + 1)
            }
          },
          {
            title: 'user',
            align: 'center',
            render: (h, params) => {
              return h('a', {
                style: {
                  'display': 'inline-block',
                  'max-width': '200px'
                },
                on: {
                  click: () => {
                    this.$router.push(
                      {
                        name: 'user-home',
                        query: {username: params.row.user.username}
                      })
                  }
                }
              }, params.row.user.username)
            }
          },
          {
            title: 'mood',
            align: 'center',
            key: 'mood'
          },
          {
            title: 'AC',
            align: 'center',
            key: 'accepted_number'
          },
          {
            title: 'Total',
            align: 'center',
            key: 'submission_number'
          },
          {
            title: 'Rating',
            align: 'center',
            render: (h, params) => {
              return h('span', utils.getACRate(params.row.accepted_number, params.row.submission_number))
            }
          }
        ]
        // options: {
        //   tooltip: {
        //     trigger: 'axis'
        //   },
        //   legend: {
        //     data: ['AC', 'Total']
        //   },
        //   grid: {
        //     x: '3%',
        //     x2: '3%'
        //   },
        //   toolbox: {
        //     show: true,
        //     feature: {
        //       dataView: {show: true, readOnly: true},
        //       magicType: {show: true, type: ['line', 'bar', 'stack']},
        //       saveAsImage: {show: true}
        //     },
        //     right: '10%'
        //   },
        //   calculable: true,
        //   xAxis: [
        //     {
        //       type: 'category',
        //       data: ['root'],
        //       axisLabel: {
        //         interval: 0,
        //         showMinLabel: true,
        //         showMaxLabel: true,
        //         align: 'center',
        //         formatter: (value, index) => {
        //           return utils.breakLongWords(value, 10)
        //         }
        //       }
        //     }
        //   ],
        //   yAxis: [
        //     {
        //       type: 'value'
        //     }
        //   ],
        //   series: [
        //     {
        //       name: 'AC',
        //       type: 'bar',
        //       data: [0],
        //       markPoint: {
        //         data: [
        //           {type: 'max', name: 'max'}
        //         ]
        //       }
        //     },
        //     {
        //       name: 'Total',
        //       type: 'bar',
        //       data: [0],
        //       markPoint: {
        //         data: [
        //           {type: 'max', name: 'max'}
        //         ]
        //       }
        //     }
        //   ]
        // }
      }
    },
    mounted () {
      this.getRankData(1)
    },
    methods: {
      init () {
        let query = this.$route.query
        this.formFilter.myself = query.myself === '1'
        this.page = parseInt(query.page) || 1
        if (this.page < 1) {
          this.page = 1
        }
        this.routeName = this.$route.name
        this.getRankData(1)
      },
      getRankData (page) {
        let offset = (page - 1) * this.limit

        // let bar = this.$refs.chart
        // bar.showLoading({maskColor: 'rgba(250, 250, 250, 0.8)'})
        this.loadingTable = true
        let myself = this.formFilter.myself === true ? '1' : '0'
        api.getUserRank(offset, this.limit, RULE_TYPE.ACM, myself).then(res => {
          this.loadingTable = false
          if (page === 1) {
            this.changeCharts(res.data.data.results.slice(0, 10))
          }
          this.total = res.data.data.total
          this.dataRank = res.data.data.results
          // bar.hideLoading()
        }).catch(() => {
          this.loadingTable = false
          // bar.hideLoading()
        })
      },
      changeCharts (rankData) {
        let [usernames, acData, totalData] = [[], [], []]
        rankData.forEach(ele => {
          usernames.push(ele.user.username)
          acData.push(ele.accepted_number)
          totalData.push(ele.submission_number)
        })
        // this.options.xAxis[0].data = usernames
        // this.options.series[0].data = acData
        // this.options.series[1].data = totalData
      },
      handleQueryChange () {
        this.page = 1
        this.changeRoute()
      },
      // 改变route， 通过监听route变化请求数据，这样可以产生route history， 用户返回时就会保存之前的状态
      changeRoute () {
        let myself = {myself: this.formFilter.myself === true ? '1' : '0'}
        let routeName = 'acm-rank'
        this.$router.push({
          name: routeName,
          query: myself
        })
      },
      goRoute (route) {
        this.$router.push(route)
      }
    },
    watch: {
      '$route' (newVal, oldVal) {
        if (newVal !== oldVal) {
          this.init()
        }
      },
      'isAuthenticated' () {
        this.init()
      }
    }
  }
</script>

<style scoped lang="less">
  .echarts {
    margin: 0 auto;
    width: 95%;
    height: 400px;
  }

  .ivu-btn-text {
    color: #57a3f3;
  }

  .flex-container {
    #main {
      flex: auto;
      margin-right: 18px;
      .filter {
        margin-right: -10px;
      }
    }
    #contest-menu {
      flex: none;
      width: 210px;
    }
  }
</style>
