<template>
    <div class="page-content">
        <div>
            <el-row :gutter="10">
                <el-col :xs="24" :sm="24" :md="24" :lg="24" :xl="12">
                    <Chart-container title="全域当天执行数据">
                        <div slot="chart">
                            <Pie id="domainProcessCount" :data-option="domainProcessCountOption"></Pie>
                        </div>
                    </Chart-container>
                </el-col>
                <el-col :xs="24" :sm="24" :md="24" :lg="24" :xl="12">
                    <Chart-container title="失败率最高的Top10域">
                        <div slot="chart">
                            <Column id="top10FailDomain" v-if="top10FailDomainOption.optionInfo" :option-info="top10FailDomainOption.optionInfo"></Column>
                        </div>
                    </Chart-container>
                </el-col>
                <el-col :xs="24" :sm="24" :md="24" :lg="24" :xl="12">
                    <Chart-container title="稳定性最差的的Top10域">
                        <div slot="chart">
                            <Column id="top10UnstableDomain" v-if="top10UnstableDomainOption.optionInfo" :option-info="top10UnstableDomainOption.optionInfo"></Column>
                        </div>
                    </Chart-container>
                </el-col>
                <el-col :xs="24" :sm="24" :md="24" :lg="24" :xl="12">
                    <Chart-container title="域版本分布">
                        <div slot="chart">
                            <Pie id="domainExecutorVersion" :data-option="domainExecutorVersionOption"></Pie>
                        </div>
                    </Chart-container>
                </el-col>
            </el-row>
        </div>
    </div>
</template>
<script>
export default {
  data() {
    return {
      zkCluster: this.$route.query.zkCluster,
      domainProcessCountOption: {},
      domainExecutorVersionOption: {},
      top10FailDomainOption: {},
      top10UnstableDomainOption: {},
    };
  },
  methods: {
    getDomainProcessCount() {
      let url = '';
      if (this.zkCluster !== '') {
        url = `/console/zkClusters/${this.zkCluster}/dashboard/domainProcessCount`;
      } else {
        url = '/console/zkClusters/dashboard/domainProcessCount';
      }
      this.$http.get(url).then((data) => {
        const resultData = JSON.parse(data);
        const error = resultData.error;
        const count = resultData.count;
        const dataArr = [['成功次数', Number.parseInt(count - error, 10)], ['失败次数', error]];
        const seriesData = [{ name: '全域当天执行数据', data: dataArr }];
        this.$set(this.domainProcessCountOption, 'seriesData', seriesData);
      })
      .catch(() => { this.$http.buildErrorHandler('获取全域当天执行数据请求失败！'); });
    },
    getTop10FailDomain() {
      let url = '';
      if (this.zkCluster !== '') {
        url = `/console/zkClusters/${this.zkCluster}/dashboard/top10FailDomain`;
      } else {
        url = '/console/zkClusters/dashboard/top10FailDomain';
      }
      this.$http.get(url).then((data) => {
        const resultData = JSON.parse(data);
        const domains = [];
        const dataArr = [];
        resultData.forEach((ele) => {
          domains.push(ele.domainName);
          this.$set(ele, 'y', ele.failureRateOfAllTime);
          this.$set(ele, 'columnType', 'domain');
          dataArr.push(ele);
        });
        const tooltip = function setTooltip() {
          return `<b>${this.point.category}</b><br/>
          错误率: ${this.point.y}<br/>
          执行总数: ${this.point.processCountOfAllTime}<br/>
          失败总数: ${this.point.errorCountOfAllTime}<br/>
          <button class="chart-tooltip-btn" onclick="vm.clearZk('/console/namespaces/${this.point.domainName}/jobAnalyse/clean')">清除zk</button>`;
        };
        const optionInfo = {
          seriesData: [{ data: dataArr }],
          xCategories: domains,
          yTitle: '失败率(小数)',
          tooltip,
        };
        this.$set(this.top10FailDomainOption, 'optionInfo', optionInfo);
      })
      .catch(() => { this.$http.buildErrorHandler('获取失败率最高的Top10域请求失败！'); });
    },
    getTop10UnstableDomain() {
      let url = '';
      if (this.zkCluster !== '') {
        url = `/console/zkClusters/${this.zkCluster}/dashboard/top10UnstableDomain`;
      } else {
        url = '/console/zkClusters/dashboard/top10UnstableDomain';
      }
      this.$http.get(url).then((data) => {
        const resultData = JSON.parse(data);
        const domains = [];
        const dataArr = [];
        resultData.forEach((ele) => {
          domains.push(ele.domainName);
          this.$set(ele, 'y', ele.shardingCount);
          this.$set(ele, 'columnType', 'domain');
          dataArr.push(ele);
        });
        const tooltip = function setTooltip() {
          return `<b>${this.point.category}</b><br/>
          分片次数: ${this.point.y}<br/>
          <button class="chart-tooltip-btn" onclick="vm.clearZk('/console/namespaces/${this.point.domainName}/shardingCount/clean')">清除zk</button>`;
        };
        const optionInfo = {
          seriesData: [{ data: dataArr }],
          xCategories: domains,
          yTitle: '分片次数',
          tooltip,
        };
        this.$set(this.top10UnstableDomainOption, 'optionInfo', optionInfo);
      })
      .catch(() => { this.$http.buildErrorHandler('获取稳定性最差的Top10域请求失败！'); });
    },
    getDomainExecutorVersionNumber() {
      let url = '';
      if (this.zkCluster !== '') {
        url = `/console/zkClusters/${this.zkCluster}/dashboard/domainExecutorVersionNumber`;
      } else {
        url = '/console/zkClusters/dashboard/domainExecutorVersionNumber';
      }
      this.$http.get(url).then((data) => {
        const arr = [];
        Object.entries(data).forEach((ele) => {
          if (ele[0] === '-1') {
            const a1 = ['未知版本', ele[1]];
            arr.push(a1);
          } else {
            arr.push(ele);
          }
        });
        const seriesData = [{ name: '该版本域数量', data: arr }];
        this.$set(this.domainExecutorVersionOption, 'seriesData', seriesData);
      })
      .catch(() => { this.$http.buildErrorHandler('获取域版本分布请求失败！'); });
    },
  },
  created() {
    this.getDomainProcessCount();
    this.getTop10FailDomain();
    this.getTop10UnstableDomain();
    this.getDomainExecutorVersionNumber();
  },
};
</script>
<style lang="sass" scoped>
</style>