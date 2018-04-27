<template>
  <div style="position: relative">
    <div class="text-center mt-5" style="margin-top: 10px;">
      <label for="autocomplete">Search for items: <span @click = "show = !show" v-show = "show" class="searchIcon"> <i class="fas fa-search"></i></span></label>
      <input v-show = "!show" class="form-control" id="autocomplete" v-model="input" style="z-index: 100;">
      <ul class="list-group">
        <li @click = "handleSearchClick(elem)" class="list-group-item" v-for="(elem, index) in matches">{{elem}}</li>
      </ul>
    </div>
    <div id="main" style="height:600px;width: 50%; margin: 0px auto"></div>
  </div>
</template>
<script>
import echarts from "echarts"
export default {
  name: "",

  data: function(){
    return {
      show: true,
      input: "",
      xaxis_able: [],
      xaxis_disable: [],
      yaxis: [],
      jsonData: [],
      markLine: {
        type: "line",
        name: "当前选择",
        markLine: {
          symbol: ["dash"],
          symbolSize: [1,1],
          itemStyle: {
            color: 'red',
            show: true,
          },
          data: [
            [
              {name: '', xAxis: 1, yAxis: 1},
              {name: '', xAxis: 0, yAxis: 1}
            ],
          ],
        }
      },
      option: {
       legend: {
         data: ['可控', '不可控', '当前选择'],
       },
       dataZoom : {
         show : true,
         orient: "vertical"
       },
       tooltip: {},
       color: ["#174775","#c2c2c2"],
       xAxis: {
         type: 'value',
         max: 'dataMax',
         axisLine: {
         show: true,
         splitLine: { show: false }
       },
       },
       yAxis: {
         type: 'category',
         triggerEvent: "true",
         data: [],
       },
       series: [
         {
           name: '可控',
           type: 'bar',
           stack: '总量',
           data: []
         },
         {
           name: '不可控',
           type: 'bar',
           stack: '总量',
           data: []
         },
       ],
     },
    }
  },
  computed: {
    matches: function(){
      var self = this;
      return this.input === ""? []:this.yaxis.filter(function(item){
        return item.indexOf(self.input) > -1? true:false;
      }).sort(function(a, b){return a.indexOf(self.input) < b.indexOf(self.input)?-1:1});
    }
  },
  methods: {
    handleSearchClick(elem){
      var myChart = echarts.init(document.getElementById('main')),
          self = this,
          index = this.yaxis.indexOf(elem);
      myChart.dispatchAction({
        type: 'dataZoom',
        startValue: index,
        endValue: index
      });
      var newMarkCoord = [
        {name: '', xAxis: 1, yAxis: index},
        {name: '', xAxis: 0, yAxis: index}
      ];
      this.input = "";
      this.show = !this.show;
      this.markLine.markLine.data.splice(0, 1, newMarkCoord);
      this.chargeSeries();
      myChart.setOption(self.option);
    },
    loadChart(){
      var self = this,
      myChart,
      options;
      myChart = echarts.init(document.getElementById('main'));
      options = this.option;
      myChart.setOption(options);
      myChart.on('click', function(param){
        self.handleClickY(param);
      });
    },
    changeChart(){
      var self = this;
      var myChart = echarts.init(document.getElementById('main'));
      var options = this.option;
      myChart.setOption(options);
    },
    chargeSeries(){
      var data_zero = {
        name: '可控',
        type: 'bar',
        stack: '总量',
        data: this.xaxis_able
      },
      data_one = {
        name: '不可控',
        type: 'bar',
        stack: '总量',
        data: this.xaxis_disable
      };
      this.option.series.splice(2, 1, this.markLine)
      this.option.yAxis.data = this.yaxis;
      this.option.series.splice(0, 1, data_zero);
      this.option.series.splice(1, 1, data_one);
    },
    handleClickY(param) {
      var value = param.value;
      if (param.componentType === "yAxis"){
        var indexY = this.yaxis.indexOf(value);
        var id = this.jsonData[indexY].id,
            newMarkCoord;
        console.log("id is: "+id);
        newMarkCoord = [
          {name: '', xAxis: 1, yAxis: indexY},
          {name: '', xAxis: 0, yAxis: indexY}
        ];
        this.markLine.markLine.data.splice(0, 1, newMarkCoord);
      }else if (param.componentType === "series") {
        console.log("value： " + value);
        if(this.xaxis_able[param.dataIndex] === 0){
          this.xaxis_able.splice(param.dataIndex, 1, value);
          this.xaxis_disable.splice(param.dataIndex, 1, 0);
        }else{
          this.xaxis_able.splice(param.dataIndex, 1, 0);
          this.xaxis_disable.splice(param.dataIndex, 1, value);
        }
      }
      this.chargeSeries();
      this.changeChart();
    },
    initChart() {
      var self = this;
      $.getJSON("../../static/test.json", function(data) {
        self.jsonData = data.charts;
        $.each(data.charts, function(i, item) {
          self.yaxis.push(item.name);
          if(item.ignored === "DISABLE"){
            self.xaxis_disable.push(item.gains);
            self.xaxis_able.push(0);
          }else{
            self.xaxis_able.push(item.gains);
            self.xaxis_disable.push(0);
          }
        });
        self.chargeSeries();
        self.loadChart();
      });
    }
  },
  mounted: function() {
    //do something after mounting vue instance
    this.initChart();
  }
}
</script>
<style scoped>
#main{
  left: 0px;
  right: 0px;
  top: 100px;
  position: absolute;
}
input, ul{
  width: 50%;
  margin: 0px auto;
}
ul, li{
  z-index: 10!important;
}
ul:hover{
  cursor: pointer;
}
.searchIcon{
  font-size: 1.5em;
  margin-left: 10px;
  top: 5px;
}
.searchIcon:hover{
  cursor: pointer;
}
</style>
