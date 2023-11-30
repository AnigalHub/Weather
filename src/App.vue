<template>
  <div id="app" :style="{background:Weather.current.background}">
    <b-container v-if="!loading">
        <h1>Weather Forecast</h1>
        <b-row>
          <b-col cols="4">
            <div class="city">{{Weather.city}}</div>
            <div class="today">
              <b-row>
                <b-col>
                  <b-row>
                    <b-col>
                      <component  :is="Weather.current.svg" class="mainSvg"/>
                    </b-col>
                    <b-col>
                      <div class="temp">{{Math.round(Weather.current.temp)}}°</div>
                    </b-col>
                  </b-row>
                  <b style="margin-left: 35%">{{Weather.current.name}}</b>
                </b-col>
                <b-col>
                  <h2>{{date.toLocaleString('en-US', { weekday: 'long' })}}</h2>
                  <div>{{date.toLocaleDateString()}}</div>
                  <div>{{date.toLocaleTimeString().slice(0,-3)}}</div>
                </b-col>
              </b-row>
            </div>
            <div class="feels">
              <b-row>
                <b-col v-for="(day, index) in Weather.current.twentyFourHours" :key="index">
                  <p><b>{{day.name}}</b></p>
                  <component :key="index" :is="day.svg" class="twentyFourHoursSvg"/>
                  <div class="twentyFourHoursTemp">{{day.temp}}</div>
                  <p class="conditions"><b>{{day.svgName}}</b></p>
                </b-col>
              </b-row>
            </div>
            <div class="feels">
              <div class="number">
                Feels like: {{Math.round(Weather.current.feel)}}°
                <br>
                Cloudy: {{Weather.current.cloudy}}%
                <br>
                Humidity: {{Weather.current.humidity}}%
                <br/>
                Precipitation: {{Weather.current.precip_mm}} mm
                <br>
                Wind: {{Weather.current.wind}} km/h
                <br/>
                Wind direction: {{Weather.current.wind_dir}}
              </div>
            </div>
          </b-col>
          <b-col>
            <div class="flex-container">
              <div class="day" v-for="(day, index) in Weather.days" :key="index">
                <b-row>
                  <b-col>
                    <h2>{{getDayOfTheWeek(index+1)}}</h2>
                  </b-col>
                  <b-col>
                    <div class="today_time" >{{day.number}}</div>
                  </b-col>
                </b-row>
                <hr/>
                    <component :key="index" :is="day.svg" width="100%" class="svgDayOfTheWeek"/>
                    <div class="darkLight">
                      <div class="light">
                        {{Math.round(day.maxTemp)}}°
                      </div>
                      <div class="light">
                        {{Math.round(day.minTemp)}}°
                      </div>
                    </div>
                <p class="conditions"><b>{{day.name}}</b></p>
              </div>
            </div>
            <WeatherChart :chartOptions="chartOptions" :chartData="chartData" type="Line"/>
          </b-col>
        </b-row>
      </b-container>
  </div>
</template>

<script>
  import axios from 'axios'

  import sunny from "./components/icons/sunny";
  import partlyCloudy from "@/components/icons/partlyCloudy";
  import partlyRainingAndSnowing from "@/components/icons/partlyRainingAndSnowing";
  import partlyRaining from "@/components/icons/partlyRaining";
  import partlySnowing from "@/components/icons/partlySnowing";
  import drizzlingRain from "./components/icons/drizzlingRain";
  import rain from "./components/icons/rain";
  import cloudy from "@/components/icons/cloudy";
  import drizzlingSnow from "@/components/icons/drizzlingSnow";
  import snowy from "@/components/icons/snowy";
  import drizzlingRainAndSnow from "@/components/icons/drizzlingRainAndSnow";
  import rainAndSnow from "@/components/icons/rainAndSnow";
  import storm from "@/components/icons/storm";
  import sunnyStorm from "@/components/icons/sunnyStorm";
  import snowStorm from "@/components/icons/snowStorm";
  import sunnySnowStorm from "@/components/icons/sunnySnowStorm";
  import overcast from "@/components/icons/overcast";
  import WeatherChart from "@/components/WeatherChart"


export default {
  name: 'App',
  components: {WeatherChart},
  data(){
    return{
      loading: true,
      chartData: {
        labels: [],
        datasets: [ { label: 'Hourly forecast', data: [] } ]
      },
      chartOptions: {
        responsive: true,
      },
      type: "Line",
      svg:partlyRaining,
      Weather:{
        city:'',
        current:{
          temp:'',
          wind:'',
          feel:'',
          humidity:'',
          twentyFourHours:[],
          svg:{},
        },
        days:[]
      },
      date:new Date(),
      Svgs:[
        {
          name:'sunny',
          svg:sunny,
          desk:'Солнечно',
          background:'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, #fff3d0 44.9%, #eaf7fc 96.1%)',
        },
        {
          name:'partly cloudy',
          svg: partlyCloudy,
          desk:'Переменная облачность',
          background: 'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, rgb(200, 223, 244) 44.9%, rgb(234, 247, 252) 96.1%)'
        },
        {
          name:'variable rain and snow',
          svg: partlyRainingAndSnowing,
          desk:'Переменный дождь и снег',
          background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, #ffe971 96.1%)'
        },
        {
          name:'variable rain',
          svg: partlyRaining,
          desk:'Переменный дождь',
          background: 'linear-gradient(179.1deg, rgb(254, 213, 102) -1.9%, rgb(200 223 244) 44.9%, rgb(234, 247, 252) 96.1%)'
        },
        {
          name:'variable snow',
          svg: partlySnowing,
          desk:'Переменный снег',
          background:'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #f3ece3 96.1%)'
        },
        {
          name:'drizzling rain',
          svg:drizzlingRain,
          desk:'Моросящий дождь',
          background:'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'rain',
          svg:rain,
          desk:'Дождь',
          background:'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'cloudy',
          svg:cloudy,
          desk:'Облачно',
          background:'linear-gradient(179.1deg, #85BCF1 -1.9%, #bedfec 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'drizzling snow',
          svg:drizzlingSnow,
          desk:'Моросящий снег',
          background:'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name:'snowy',
          svg:snowy,
          desk:'Снежно',
          background:'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name:'drizzling rain and snow',
          svg:drizzlingRainAndSnow,
          desk:'Моросящий дождь и снег',
          background:'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name:'rain and snow',
          svg:rainAndSnow,
          desk:'Дождь и снег',
          background:'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name:'storm',
          svg:storm,
          desk:'Гроза',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'sunnyStorm',
          svg:sunnyStorm,
          desk:'Гроза и солнце',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #dedbd1 96.1%)'
        },
        {
          name:'snowStorm',
          svg:snowStorm,
          desk:'Гроза и снег',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'sunnySnowStorm',
          svg:sunnySnowStorm,
          desk:'Гроза, солнце и снег',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'overcast',
          svg:overcast,
          desk:'Пасмурно',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
      ],
      IncomingPictures:[
        {
          codes:[1000]
        },
        {
          codes:[1003]
        },
        {
          codes:[1069,1252]
        },
        {
          codes:[1063, 1192, 1243, 1246]
        },
        {
          codes:[1066, 1255, 1258, 1261, 1264]
        },
        {
          codes:[1150, 1153, 1180, 1183, 1186, 1189, 1240]
        },
        {
          codes:[1072, 1171, 1195]
        },
        {
          codes:[1006]
        },
        {
          codes:[1114, 1117, 1168, 1204,1210, 1213, 1216, 1219]
        },
        {
          codes:[1207, 1222, 1225]
        },
        {
          codes:[1198, 1237]
        },
        {
          codes:[1201, 1249]
        },
        {
          codes:[1276]
        },
        {
          codes:[1087, 1273]
        },
        {
          codes:[1282]
        },
        {
          codes:[1279]
        },
        {
          codes:[1009,1030, 1135, 1147]
        },
      ]
    }
  },
  methods:{
    async getWeather(){
      const API_KEY = "fcdf7a43f2dc4f7e800180856230711"
      let q = 'Moscow';
      let days = '7'
      let response = await axios.get(`http://api.weatherapi.com/v1/forecast.json`,
              {
                params: {
                  key:API_KEY,
                  q,
                  days
                }
              });
      return response;
    },
    getDayMs(i){
      let date = new Date();
      return date.setDate(date.getDate() + i);
    },
    getDate(i){
      let day = this.getDayMs(i);
      return new Date(day).toLocaleDateString();
    },
    getDayOfTheWeek(i){
      let day = this.getDayMs(i);
      return new Date(day).toLocaleString('en-US', { weekday: 'long' });
    },
    getCurrentData(data){
      let current =  data.current;
      let forecast = data.forecast;
      this.Weather.city = data.location.name;
      this.Weather.current.temp = current.temp_c;
      this.Weather.current.wind = current.wind_kph;
      this.Weather.current.feel = current.feelslike_c;
      this.Weather.current.humidity = current.humidity;
      this.Weather.current.svg = this.putInSvg(current.condition.code)[0];
      this.Weather.current.name = this.putInSvg(current.condition.code)[1];
      this.Weather.current.background = this.putInSvg(current.condition.code)[2];
      this.Weather.current.precip_mm = current.precip_mm;
      this.Weather.current.wind_dir = current.wind_dir;
      this.Weather.current.cloudy = current.cloud;


      for(let i=6; i < forecast.forecastday[0].hour.length; i=i+6){
        let time = {};

        time.temp = forecast.forecastday[0].hour[i].temp_c;
        let times = forecast.forecastday[0].hour[i].time.substring(10).trim();

        if(times === '06:00' && i === 6){
          time.name = 'Morning'
        }
        if(times === '12:00' && i === 12){
          time.name = 'Afternoon'
        }
        if(times === '18:00' && i === 18){
          time.name = 'Evening'
        }
        let code = forecast.forecastday[0].hour[i].condition.code;
        time.svg = this.putInSvg(code)[0];
        time.svgName = this.putInSvg(code)[1];
        this.Weather.current.twentyFourHours.push(time);
      }
      console.log('this.Weather.current.twentyFourHours', this.Weather.current.twentyFourHours)
    },
    putInSvg(codePicture){
      for(const i in this.IncomingPictures) {
        for(const code of this.IncomingPictures[i].codes) {
          if(code === codePicture){
            return [this.Svgs[i].svg, this.Svgs[i].name, this.Svgs[i].background ];
          }
        }
      }
    },
    getWeekData(data){
      let forecast = data.forecast;
      for(let jj=1; jj < forecast.forecastday.length; jj++){
        let day = {};
        day.maxTemp = forecast.forecastday[jj].day.maxtemp_c;
        day.minTemp = forecast.forecastday[jj].day.mintemp_c;
        let code = forecast.forecastday[jj].day.condition.code;
        day.svg = this.putInSvg(code)[0];
        day.name = this.putInSvg(code)[1];
        day.number = new Date(this.getDayMs(jj)).toLocaleDateString();

        // this.chartData.labels.push(day.number);
        // this.chartData.datasets[0].data.push(Math.round(day.minTemp));
        this.Weather.days.push(day);

      }
    },
  },
  async created() {
    let response  = await this.getWeather()
    this.loading = false;
    console.log('created', response.data);
    this.getCurrentData(response.data);
    this.getWeekData(response.data);
    console.log('this.chartData', this.chartData);
    for(let i=0; i < response.data.forecast.forecastday[0].hour.length; i++){
      let time = (response.data.forecast.forecastday[0].hour[i].time).substring(10).trim();
      let temp_c = Math.round(response.data.forecast.forecastday[0].hour[i].temp_c);
      this.chartData.labels.push(time);
      this.chartData.datasets[0].backgroundColor = 'rgb(133, 188, 241)'
      this.chartData.datasets[0].data.push(temp_c);
    }
  },


}
</script>

<style lang="scss">
  .twentyFourHoursSvg{
    width: 100%;
    margin-top: -20%;
  }
  .twentyFourHoursTemp{
    font-size: 1.6rem;
    text-align: center;
    margin-top: -10%;
  }
  h1{
    margin-bottom: 1rem !important;
    text-shadow: 1.5px 1.5px 1.5px #f3e0e0;
  }
  #app {
    //background: linear-gradient(179.1deg, rgb(203 180 180) -1.9%, rgb(187 153 152) 44.9%, rgb(127 147 184) 96.1%);
    //background: linear-gradient(179.1deg, #fff -1.9%, #D5F3FF 44.9%, #85BCF1 96.1%);
    background: linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%);
    min-height: 100vh;
    text-align: center;
    padding: 45px 0;
    font-family: 'Roboto Condensed', sans-serif;
  }
  canvas{
    background: rgba(255, 253, 253, 0.36);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    //height: 325px !important;
    margin: 0 0 0 0;
  }
  .today{
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    font-size: 1.4rem;
    text-align: left;
    color: black;
    padding: 15px;
    margin-bottom: 3%;
   // height: 75vh;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
   // height: 65.7vh;
  }
  .feels{
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    margin-bottom: 3%;
    padding: 15px;
    // height: 75vh;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    // height: 65.7vh;
  }
  .today_time{
    color: black;
    text-align: left;
    font-size: 1.3rem;
  }
  hr{
    margin-top: .25rem !important;
    margin-bottom: .25rem !important;
    background: transparent;
    box-shadow: 0 .5px .5px rgba(0, 0, 0, 0.25);
  }
  h2{
    text-align: left;
    color: black;
    font-weight: 500 !important;
  }
  .now{
    font-size: 2.5rem;
    color: black;
    text-align: center;
  }
  .temp{
    font-size: 3rem;
    line-height: 2rem;
    margin-top: 40%;
    //margin-top: 55%;
   margin-left: 10%;
  }
  .city{
    font-size: 2rem;
    line-height: 2rem;
    font-weight: 500;
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    margin-bottom: 3%;
  }
  .mainSvg{
    margin: 40% auto 0 -15%;
    width: 230%;
  }
  .svgDayOfTheWeek{
   // margin-left: -10px;
  }
  .number{
    font-size: 1.2rem;
    text-align: left;
    color: black;
  }
  .light, .dark{
   // width: 52px;
    height: 35px;
   // background: #f1fbff;
    //padding: 5px;
    font-size: 1.6rem;
    text-align: center;
   // border-radius: 10px;
    //box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  .conditions{
    font-size: 0.85rem !important;
    line-height: 1rem;
    text-align: center;
    margin-bottom: 0 !important;
  }
  .dark{
    margin-left: 20px;
  }
  .light{
    margin-top: -5px;
  }
  .day{
    background: rgb(255 253 253 / 36%);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    h2{
      font-size: 1rem !important;
      font-weight: 500;
      margin-bottom: 0 !important;
    }
    .today_time{
      font-size: .9rem
    }
  }
  .flex-container > .day{
    width: 15.16%;
    //height: 29vh;
    margin: 0 0.75% 1%;
  }
  .flex-container{
    display: flex;
    flex-wrap: wrap;
  }
  @media screen and (min-width: 768px) and (max-width: 992px){
    .today{
      height: 65vh;
    }
    .flex-container > .day {
      width: 48%;
      height: 20.8vh;
    }
    .day hr{
      margin-bottom: 15px !important;
    }
    h2{
      font-size: 1.6rem !important;
    }
    .today_time,.number {
      font-size: 1.2rem;
    }
    .now,.temp{
      font-size: 2rem;
    }
    .day h2 {
      font-size: 1.2rem !important;
    }
    .light, .dark {
      width: 42px;
      height: 38px;
      font-size: 1.35rem;
    }
    .darkLight{
      margin-left: -20px;
    }
  }
</style>
