<template>
  <div class="chart">
    <div class="chart__body">
    </div>
  </div>
</template>

<style lang="scss">
  .chart {
    width: $px496;
    padding: $px8;
    margin-bottom: $px24;

    border: $px1 solid $grey;

    /* HACK Since apex charts doesn't support showing toolbar on hover
    let's make a hack: */
    .apexcharts-toolbar {
      display: none !important;
    }

    &:hover .apexcharts-toolbar {
      display: flex !important;
    }
  }
</style>

<script>
import DashboardStore from '../stores/DashboardStore';
import DashboardConfig from '../configs/DashboardConfig';
import ApexChartConfig from '../configs/ApexChartConfig';

export default {
  name: 'Chart',
  data() {
    return {
      dashboardStore: DashboardStore.data,
      dashboardConfig: DashboardConfig,
      apexChartConfig: ApexChartConfig,
      chart: null,
    };
  },
  props: [
    'metric',
    'deviceType',
    'url',
    'startDate',
    'finishDate',
  ],
  beforeMount() {
    window.Apex = this.apexChartConfig;
    window.ApexCharts = ApexCharts;
  },
  mounted() {
    this.init();
    console.log(this.timeline());
  },
  render(createElement) {
    return createElement('div');
  },
  watch: {
    startDate() {
      this.chart.updateOptions({
        xaxis: {
          min: this.startDate.getTime(),
          max: this.finishDate.getTime(),
        },
        yaxis: {
          min: 0,
          max: this.maxValue(),
        },
      });
    },
    finishDate() {
      this.chart.updateOptions({
        xaxis: {
          min: this.startDate.getTime(),
          max: this.finishDate.getTime(),
        },
        yaxis: {
          min: 0,
          max: this.maxValue(),
        },
      });
    },
    url() {
      if (!this.chart) {
        this.init();
      } else {
        this.chart.updateSeries([{
          data: this.timeline(),
        }]);

        this.chart.updateOptions({
          xaxis: {
            min: this.startDate.getTime(),
            max: this.finishDate.getTime(),
          },
          yaxis: {
            min: 0,
            max: this.maxValue(),
          },
        });
      }
    },
  },
  methods: {
    timeline() {
      const { deviceType } = this;
      const { metric } = this;
      const seriesData = this.dashboardStore.urlData[this.url][deviceType];
      return Object.keys(seriesData).map((timestamp) => {
        const newDate = timestamp.slice(0, -6);
        console.log(newDate);
        return [new Date(newDate), seriesData[timestamp][metric]];
      });
    },
    apexChartOptions() {
      let optionsLine = {
        chart: {
          id: `${this.deviceType}-${this.metric}`,
          type: 'line',
          toolbar: {
            show: true,
            tools: {
              download: true,
              selection: true,
              zoom: true,
              zoomin: true,
              zoomout: true,
              pan: false,
              reset: true,
            },
          },
        },
        title: {
          text: `${this.deviceType} - ${this.metric}`,
        },
        colors: ['#008FFB'],
        series: [{
          data: this.timeline(),
        }],
        yaxis: {
          min: 0,
          max: this.maxValue(),
          labels: {
            minWidth: 10,
          },
          decimalsInFloat: 1,
        },
      };

      const metricConfig = { ...this.dashboardConfig.chartOptions[this.metric] };
      if (metricConfig) {
        optionsLine = {
          ...optionsLine,
          ...metricConfig,
        };
      }

      return optionsLine;
    },
    maxValue() {
      if (this.dashboardConfig.chartOptions[this.metric].yaxis) {
        return this.dashboardConfig.chartOptions[this.metric].yaxis.max;
      }
      const timeline = this.timeline();
      const values = timeline.map((value, _) => value[1]);

      return Math.ceil(Math.max(...values) + 1);
    },
    init() {
      this.chart = new ApexCharts(this.$el, this.apexChartOptions());
      this.chart.render();
    },
  },
};
</script>
