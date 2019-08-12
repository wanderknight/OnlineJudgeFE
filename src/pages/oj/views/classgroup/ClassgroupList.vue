<template>
  <div>
    <h1>classgroup-list</h1>
    <ol id="classgroup-list">
      <li v-for="classgroup in classgroups" :key="classgroup.title">
        <Row type="flex" justify="space-between" align="middle">
          <p >
            {{classgroup.title}}
          </p>
        </Row>
      </li>
    </ol>
  </div>
</template>

<script>
  import api from '@oj/api'

  const limit = 8

  export default {
    name: 'classgroup-list',
    data () {
      return {
        page: 1,
        query: {
          status: '',
          keyword: '',
          rule_type: ''
        },
        limit: limit,
        total: 0,
        rows: '',
        classgroups: []
//      CONTEST_STATUS_REVERSE: CONTEST_STATUS_REVERSE,
//      for password modal use
//      cur_contest_id: ''
      }
    },
    beforeRouteEnter (to, from, next) {
      api.getClassgroupList(0, limit).then((res) => {
        next((vm) => {
          vm.classgroups = res.data.data.results
          vm.total = res.data.data.total
        })
      }, (res) => {
        next()
      })
    },
    methods: {
      init () {
        let route = this.$route.query
        this.query.status = route.status || ''
        this.query.rule_type = route.rule_type || ''
        this.query.keyword = route.keyword || ''
        this.page = parseInt(route.page) || 1
        this.getClassgroupList()
      },
      getClassgroupList (page = 1) {
        let offset = (page - 1) * this.limit
        api.getClassgroupList(offset, this.limit).then((res) => {
          this.classgroups = res.data.data.results
          this.total = res.data.data.total
        })
      }
    }
  }
</script>

<style lang="less" scoped>
/*  #classgroup-card {
    #keyword {
      width: 80%;
      margin-right: 30px;
    }
    #no-classgroup {
      text-align: center;
      font-size: 16px;
      padding: 20px;
    }
    #classgroup-list {
      > li {
        padding: 20px;
        border-bottom: 1px solid rgba(187, 187, 187, 0.5);
        list-style: none;

        .trophy {
          height: 40px;
          margin-left: 10px;
          margin-right: -20px;
        }
        .classgroup-main {
          .title {
            font-size: 18px;
            a.entry {
              color: #495060;
              &:hover {
                color: #2d8cf0;
                border-bottom: 1px solid #2d8cf0;
              }
            }
          }
          li {
            display: inline-block;
            padding: 10px 0 0 10px;
            &:first-child {
              padding: 10px 0 0 0;
            }
          }
        }
      }
    }
  }*/
</style>
