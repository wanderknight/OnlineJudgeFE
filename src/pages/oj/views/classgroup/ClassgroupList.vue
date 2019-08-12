<template>
  <h1>class list</h1>
  <ol id="classgroup-list">
    <li v-for="classgroup in classgroups" :key="classgroup.title">
      <Row type="flex" justify="space-between" align="middle">
        <img class="trophy" src="../../../../assets/Cup.png"/>
        <Col :span="18" class="contest-main">
          <p class="title">
            <a class="entry" @click.stop="goContest(contest)">
              {{contest.title}}
            </a>
            <template v-if="contest.contest_type != 'Public'">
              <Icon type="ios-locked-outline" size="20"></Icon>
            </template>
          </p>
          <ul class="detail">
            <li>
              <Icon type="calendar" color="#3091f2"></Icon>
              {{contest.start_time | localtime('YYYY-M-D HH:mm') }}
            </li>
            <li>
              <Icon type="android-time" color="#3091f2"></Icon>
              {{getDuration(contest.start_time, contest.end_time)}}
            </li>
            <li>
              <Button size="small" shape="circle" @click="onRuleChange(contest.rule_type)">
                {{contest.rule_type}}
              </Button>
            </li>
          </ul>
        </Col>
        <Col :span="4" style="text-align: center">
          <Tag type="dot" :color="CONTEST_STATUS_REVERSE[contest.status].color">
            {{CONTEST_STATUS_REVERSE[contest.status].name}}
          </Tag>
        </Col>
      </Row>
    </li>
  </ol>
</template>

<script>
  import api from '@oj/api'

  export default {
    name: 'classgroup-list',
    data() {
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
        classgroups: [],
        //CONTEST_STATUS_REVERSE: CONTEST_STATUS_REVERSE,
//      for password modal use
        //cur_contest_id: ''
      }
    },
    methods: {
      init() {
        let route = this.$route.query
        this.query.status = route.status || ''
        this.query.rule_type = route.rule_type || ''
        this.query.keyword = route.keyword || ''
        this.page = parseInt(route.page) || 1
        this.getClassgroupsList()
      },
      getClassgroupsList(page = 1) {
        let offset = (page - 1) * this.limit
        api.getClassgroupsList(offset, this.limit, this.query).then((res) => {
          this.contests = res.data.data.results
          this.total = res.data.data.total
        })
      },
    }
  }
</script>

<style scoped>

</style>
