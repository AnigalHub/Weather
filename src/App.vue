<template>
  <div id="app">
    <b-container>
        <h1>Weather Forecast</h1>
        <b-row>
          <b-col cols="4">
            <div class="today">
              <h2>{{date.toLocaleString('en-US', { weekday: 'long' })}}</h2>
              <hr>
              <b-row class="today_time">
                <b-col >
                  {{date.toLocaleDateString()}}
                </b-col>
                <b-col cols="4">
                  {{date.toLocaleTimeString().slice(0,-3)}}
                </b-col>
              </b-row>
              <hr>
              <component  :is="Weather.current.svg" width="95%"/>
              <div class="now">
                <div class="temp">{{Math.round(Weather.current.temp)}}°</div>
                <div>{{Weather.city}}</div>
              </div>
              <hr>
              <div class="number">
                Feels like: {{Math.round(Weather.current.feel)}}°
                <br>
                Humidity: {{Weather.current.humidity}}%
                <br>
                Wind: {{Weather.current.wind}} km/h
              </div>
            </div>
          </b-col>
          <b-col>
            <div class="flex-container" >
              <div class="day" v-for="(day, index) in Weather.days" :key="index">
                <b-row>
                  <b-col>
                    <h2>{{getDayOfTheWeek(index+1)}}</h2>
                  </b-col>
                  <b-col>
                    <div class="today_time" >{{getDate(index+1)}}</div>
                  </b-col>
                </b-row>
                <hr/>
                <b-row>
                  <b-col>
                    <component :key="index" :is="day.svg" width="105%" class="svgDayOfTheWeek"/>
                  </b-col>
                  <b-col cols="5">
                    <div class="darkLight">
                      <div class="dark">
                        {{Math.round(day.maxTemp)}}°
                      </div>
                      <div class="light">
                        {{Math.round(day.minTemp)}}°
                      </div>
                    </div>
                  </b-col>
                </b-row>
               <b>{{day.name}}</b>
              </div>
            </div>
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

export default {
  name: 'App',
  components: {},
  data(){
    return{
      svg:partlyRaining,
      Weather:{
        city:'',
        current:{
          temp:'',
          wind:'',
          feel:'',
          humidity:'',
          svg:{},
        },
        days:[]
      },
      date:new Date(),
      Svgs:[
        {
          name:'sunny',
          svg:sunny,
          desk:'Солнечно'
        },
        {
          name:'partly cloudy',
          svg: partlyCloudy,
          desk:'Переменная облачность'
        },
        {
          name:'variable rain and snow',
          svg: partlyRainingAndSnowing,
          desk:'Переменный дождь и снег'
        },
        {
          name:'variable rain',
          svg: partlyRaining,
          desk:'Переменный дождь'
        },
        {
          name:'variable snow',
          svg: partlySnowing,
          desk:'Переменный снег'
        },
        {
          name:'drizzling rain',
          svg:drizzlingRain,
          desk:'Моросящий дождь'
        },
        {
          svg:rain,
          desk:'Дождь'
        },
        {
          name:'cloudy',
          svg:cloudy,
          desk:'Облачно'
        },
        {
          name:'drizzling snow',
          svg:drizzlingSnow,
          desk:'Моросящий снег'
        },
        {
          name:'snowy',
          svg:snowy,
          desk:'Снежно'
        },
        {
          name:'drizzling rain and snow',
          svg:drizzlingRainAndSnow,
          desk:'Моросящий дождь и снег'
        },
        {
          name:'rain and snow',
          svg:rainAndSnow,
          desk:'Дождь и снег'
        },
        {
          name:'storm',
          svg:storm,
          desk:'Гроза'
        },
        {
          name:'sunnyStorm',
          svg:sunnyStorm,
          desk:'Гроза и солнце'
        },
        {
          name:'snowStorm',
          svg:snowStorm,
          desk:'Гроза и снег'
        },
        {
          name:'sunnySnowStorm',
          svg:sunnySnowStorm,
          desk:'Гроза, солнце и снег'
        },
        {
          name:'overcast',
          svg:overcast,
          desk:'Пасмурно'
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
      this.Weather.city = data.location.name;
      this.Weather.current.temp = current.temp_c;
      this.Weather.current.wind = current.wind_kph;
      this.Weather.current.feel = current.feelslike_c;
      this.Weather.current.humidity = current.humidity;
      this.Weather.current.svg = this.putInSvg(current.condition.code)[0]
    },
    putInSvg(codePicture){
      for(const i in this.IncomingPictures) {
        for(const code of this.IncomingPictures[i].codes) {
          if(code === codePicture){
            return [this.Svgs[i].svg, this.Svgs[i].name ];
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
        this.Weather.days.push(day);
      }
    }
  },
  async created() {
    let response  = await this.getWeather()
    console.log('created', response.data);
    this.getCurrentData(response.data);
    this.getWeekData(response.data)
  }
}
</script>

<style lang="scss">
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
  .today{
    background: rgb(255 253 253 / 36%);
    border-radius: 10px;
    padding: 15px;
    height: 75vh;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  .today_time{
    color: black;
    text-align: left;
    font-size: 1.3rem;
  }
  hr{
    margin-top: .5rem !important;
    margin-bottom: .5rem !important;
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
    font-size: 3.5rem;
    line-height: 3.5rem;
  }
  .svgDayOfTheWeek{
    margin-left: -10px;
  }
  .number{
    font-size: 1.3rem;
    text-align: left;
    color: black;
  }
  .light, .dark{
    width: 52px;
    height: 45px;
    background: #f1fbff;
    padding: 5px;
    font-size: 1.8rem;
    text-align: center;
    border-radius: 10px;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
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
      font-size: 1.3rem !important;
      font-weight: 500;
      margin-bottom: 0 !important;
    }
    .today_time{
      font-size: 1rem
    }
  }
  .flex-container > .day{
    width: 43%;
    height: 23.6vh;
    margin: 0 1% 2%;
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
