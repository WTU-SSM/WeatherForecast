<template>
  <div id="app">
    <div id="bg">
      <span></span>
    </div>

      <!-- 搜索框 -->
      <div id="select" >
        <input type="text" :value="city" id="cityInput" @keyup.enter="updateCity" />
        <img src="~assets/img/search.svg" @click="updateCity" />
      </div>
      <div id="weather">

        <!-- 切换城市和主要天气 -->
        <div id="weather-today">
          <div id="position-and-date">
            <span >{{ city }}</span> 
            <span >[切换城市]</span>
            <p>
              <span class="marginRight">{{ date[whichday] }}</span>
              <span>{{ week[whichday] }}</span>
            </p>
          </div>

          <div id="temperature">

            <!-- 天气 -->
            <div id="main-temperature">
              <!-- 主要天气的图标 -->
              <img :src="iconTodayPath" alt="">
              <span id="temperature-number">{{ tem[whichday] }}</span>
              <div>
                <p>{{ wea[whichday] }}</p>
                <p>{{ temMin[whichday] }} ~ {{ temMax[whichday] }}</p>
              </div>
            </div>

            <!-- 其他数据 -->
            <div>
              <span class="marginRight">风向：{{ win[whichday] }} </span>
              <span>风速：{{ winSpeed[whichday] }}</span>
            </div>

          </div>

        </div>

        <!-- 未来天气 -->
        <p id="broadcast-title" >{{ city }}天气预报一周</p>

        <!-- 未来天气数据 -->
        <div id="weather-fature-week">
          <div v-for="(item, index) in date" :key="item.id" id="weather-fature-day" @click="changewhichday(index)">
            <div>
              <p>{{ week[index] }}</p>
              <p>{{ date[index] }}</p>
              <img :src="getIconPath(index)" alt="">
              <p>{{ temMin[index] }} ~ {{ temMax[index] }}</p>
              <p>{{ wea[index] }}</p>
              <p>{{ win[index] }}</p>
            </div>
          </div>
        </div>

        <!-- 天气折线图 -->
        <div id="chart"></div>
      </div>
    </div>
  <!-- </div> -->
</template>

<script>
import {request} from "./network/request";
let echarts = require("echarts");
let myChart;

export default {
  name: "App",
  components: {},
  data() {
    return {
      city: "武汉",
      // 未来七日数据
      date: [],             // 日期
      week: [],             // 星期
      wea: [],              // 天气
      tem: [],              // 平均温度
      temMax: [],           // 最高温
      temMin: [],           // 最低温
      win: [],              // 风向 
      winSpeed: [],         // 风速
      wea_img: [],          // 气象图标

      option: {},         
      whichday: 0,        
      
    };
  },
  methods: {
    updateCity() {
      this.selectStateChange()    
      this.cityChange()         
      this.getWeatherInfoAndDrawChart() 
    },

    getWeatherInfoAndDrawChart() {
      // 网络请求
      request({
        url: "",
        method: "get",
        params: {
          appid: "22266748",
          appsecret: "pTIzTk6N ",
          version: "v9",
          city: this.city,
          vue: "1",
        },
      }).then(res => {
        console.log(res);
        // 获取未来七天的数据
        for (let i = 0; i < res.data.length; i++) {
          this.$set(this.date,i,res.data[i].date)                                           
          this.$set(this.week,i,res.data[i].week)                                              
          this.$set(this.wea,i,res.data[i].wea)                                 
          this.$set(this.tem,i,res.data[i].tem)                                           
          this.$set(this.temMax,i,res.data[i].tem1)                                        
          this.$set(this.temMin,i,res.data[i].tem2)                                        
          this.$set(this.win,i,res.data[i].win[0])                               
          this.$set(this.wea_img,i,res.data[i].wea_img)                                
          this.$set(this.winSpeed,i,res.data[i].win_speed)             
        }
        this.drawChart(this.week,this.tem)
      })
    },

    selectStateChange() {    
      this.showSelect = !this.showSelect;
    },

    cityChange() {            
      this.city = document.getElementById("cityInput").value; 
    },

    changewhichday(index) {   
      this.whichday = index;
    },

    drawChart(paramX, paramY) {             
      console.log("paramX",paramX);
      console.log("paramY",paramY);
      

        let test = this.chartTem = this.tem.map(curValue => {
          return curValue.slice(0, -1);
        });

     
        this.option = {

        color: ["#ffffff"],

          grid: {
            top: 10,
            bottom: 40,
            left: 30,
            right: 30,
          },

          xAxis: [  
            {
              type: "category",
              data: paramX,   
              show: true,
              axisPointer: {
                show: true,
              },
              axisLine: {
                show: false,
              },
              axisLabel: {
                color: "rgba(255,255,255,1)",
              },
            },
          ],

          yAxis: [
            {
              type: "value",
              show: true,
              axisPointer: {
                show: true,
              },
              axisLine: {
                show: false,
              },
              axisLabel: {
                color: "rgba(255,255,255,1)",
              },
            },
          ],

          series: [
            {
              data: paramY, 
              type: "line",
            },
          ],
        };
      
      myChart.setOption(this.option);
    },

    getIconPath(index) {                              
      if(this.wea_img[index] === undefined){       
        return require("assets/img/icon/lei.svg")
      }
      return require("assets/img/icon/"+this.wea_img[index]+".svg")
    },
  },

  computed: {
    iconTodayPath() {
      if(this.wea_img.length == 0) {      
        return require("assets/img/icon/leiToday.svg")
      }
      return require("assets/img/icon/"+this.wea_img[this.whichday]+"Today.svg")
    },
  },

  mounted() {   
    myChart = echarts.init(document.getElementById("chart")); 
    this.updateCity(); 
  },
};
</script>

<style>
@import "assets/css/weatherforecast.css";
</style>
