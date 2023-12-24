<template>
  <div id="app" :style="{background:Weather.current.background}">
    <b-container v-if="!loading && show">
      <div class="city">
        <span>Choose one of the options: </span>
        <select v-model="selected" @click="recalculateTheWeather()">
        <option v-for="option in options" v-bind:value="option.text">
          {{ option.text }}
        </option>
      </select>
      </div>
      <b-row class="mainRow">
          <b-col cols="3">
            <div>
              <h4>Today:</h4>
              <div class="today line">
                <b-row>
                  <b-col cols="5">
                    <component  :is="Weather.current.svg" class="mainSvg"/>
                    <p class="temp">{{Math.round(Weather.current.temp)}}°</p>
                    <p class="conditions"><b>{{Weather.current.name}}</b></p>
                  </b-col>
                  <b-col class="today_time">
                    <p><b>{{date.toLocaleString('en-US', { weekday: 'long' })}}</b></p>
                    <hr/>
                    <p>{{date.toLocaleDateString()}}</p>
                    <p>{{date.toLocaleTimeString().slice(0,-3)}}</p>
                    <hr/>
                    <p>Feels like: {{Math.round(Weather.current.feel)}}°</p>
                  </b-col>
                </b-row>
              </div>
            </div>
            <div>
              <h4>Detailed description:</h4>
              <div class="feels">
                <div class="flex-container">
                  <p>Cloudy: <b>{{Weather.current.cloudy}}%</b></p>
                  <p>Humidity: <b>{{Weather.current.humidity}}%</b></p>
                  <p>Precipitation: <b>{{Weather.current.precip_mm}} mm</b></p>
                  <p>Wind: <b>{{Weather.current.wind}} km/h</b></p>
                  <p>Visibility:<b>{{Weather.current.vis_km}} km</b></p>
                  <p>Wind direction by compass: <b>{{Weather.current.wind_dir}}</b></p>
                  <p>Wind direction in degree: <b>{{Weather.current.wind_degree}}</b></p>
                </div>
              </div>
            </div>
            <div>
              <h4>Twenty-four hours:</h4>
              <div class="hours">
                <div v-for="(day, index) in Weather.current.twentyFourHours" :key="index">
                  <p class="text-center"><b>{{day.name}}</b></p>
                  <component :key="index" :is="day.svg" class="twentyFourHoursSvg"/>
                  <div class="twentyFourHoursTemp">{{Math.round(day.temp)}}°</div>
                  <p class="conditions"><b>{{day.svgName}}</b></p>
                </div>
              </div>
            </div>
          </b-col>
          <b-col>
            <b-row class="addRow">
              <b-col>
                  <h4>Light and dark time of day:</h4>
                  <div class="flex-container">
                    <div class="day">
                      <p class="day_name">Light time of day:</p>
                      <hr/>
                      <b-row class="darkLight">
                        <b-col cols="4">
                          <component :is="SvgsDay[0].svg" class="sunMoonSvg"/>
                        </b-col>
                        <b-col>
                          <p class="sunMoon conditions"><b> from {{Weather.current.sunMoon.start}} to {{Weather.current.sunMoon.end}}</b></p>
                          <p class="conditions"><b>{{Weather.current.sunMoon.sun_phase}}</b></p>
                        </b-col>
                      </b-row>
                    </div>
                    <div class="day">
                      <p class="day_name">Dark time of day:</p>
                      <hr/>
                      <b-row class="darkLight">
                        <b-col cols="4">
                          <component :is="SvgsNight[0].svg" class="sunMoonSvg"/>
                        </b-col>
                        <b-col>
                          <p class="sunMoon conditions"><b> from {{Weather.current.sunMoon.end}} to {{Weather.current.sunMoon.start}}</b></p>
                          <p class="conditions"><b>{{Weather.current.sunMoon.moon_phase}}</b></p>
                        </b-col>
                      </b-row>
                    </div>
                  </div>
              </b-col>
              <b-col>
                <div>
                  <h4>Coming days:</h4>
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
                      <b-row class="darkLight">
                        <b-col cols="5">
                          <component :key="index" :is="day.svg" width="100%" class="svgDayOfTheWeek"/>
                        </b-col>
                        <b-col>
                          <p class="light">{{Math.round(day.minTemp)}}°   {{Math.round(day.maxTemp)}}°</p>
                          <p class="conditions"><b>{{day.name}}</b></p>
                        </b-col>
                      </b-row>
                    </div>
                  </div>
                </div>
              </b-col>
            </b-row>
            <h4 class="additionally_h">Hourly forecast</h4>
            <div class="flex-container">
              <div class="Line"><WeatherChart :chartOptions="chartOptions" :chartData="tempDay" type="Line"/></div>
              <div class="Line"><WeatherChart :chartOptions="chartOptions" :chartData="rainSnowDay" type="Line"/></div>
              <div class="Line"><WeatherChart :chartOptions="chartOptions" :chartData="cloudDay" type="Line"/></div>
              <div class="Line"><WeatherChart :chartOptions="chartOptions" :chartData="humidityDay" type="Line"/></div>
            </div>
          </b-col>
        </b-row>
    </b-container>
    <b-container v-else class="container_error">
      <div class="error">
        <component :is="svgError"/>
        <h1>Error</h1>
        <h2>An error has occurred. We apologize. We will fix it as soon as possible and are sincerely sorry for any inconvenience caused.</h2>
      </div>
    </b-container>
  </div>
</template>

<script>
  import axios from 'axios'

  import error from "./components/icons/error";
  import sunny from "./components/icons/sunny";
  import moon from "./components/icons/moon";
  import partlyCloudy from "@/components/icons/partlyCloudy";
  import partlyCloudyMoon from "@/components/icons/partlyCloudyMoon";
  import partlyRainingAndSnowing from "@/components/icons/partlyRainingAndSnowing";
  import partlyRainingAndSnowingMoon from "@/components/icons/partlyRainingAndSnowingMoon";
  import partlyRaining from "@/components/icons/partlyRaining";
  import partlyRainingMoon from "@/components/icons/partlyRainingMoon";
  import partlySnowing from "@/components/icons/partlySnowing";
  import partlySnowingMoon from "@/components/icons/partlySnowingMoon";
  import drizzlingRain from "./components/icons/drizzlingRain";
  import rain from "./components/icons/rain";
  import cloudy from "@/components/icons/cloudy";
  import drizzlingSnow from "@/components/icons/drizzlingSnow";
  import snowy from "@/components/icons/snowy";
  import drizzlingRainAndSnow from "@/components/icons/drizzlingRainAndSnow";
  import rainAndSnow from "@/components/icons/rainAndSnow";
  import storm from "@/components/icons/storm";
  import sunnyStorm from "@/components/icons/sunnyStorm";
  import moonStorm from "@/components/icons/moonStorm";
  import snowStorm from "@/components/icons/snowStorm";
  import sunnySnowStorm from "@/components/icons/sunnySnowStorm";
  import moonSnowStorm from "@/components/icons/moonSnowStorm";
  import overcast from "@/components/icons/overcast";
  import fog from "@/components/icons/fog";
  import WeatherChart from "@/components/WeatherChart"

export default {
  name: 'App',
  components: {WeatherChart},
  data(){
    return{
      svgError: error,
      loading: true,
      show: true,
      selected: 'Moscow',
      options: [
        { text: 'Moscow'},
        { text: 'Kolomna' },
        { text: 'Ryazan'},
        { text: 'Ryazhsk'},
        { text: 'Vladimir'},
      ],
      tempDay: {
        labels: [],
        datasets: [
          { backgroundColor:'rgb(133, 188, 241)', label: 'Temperature', data: [] },
          { backgroundColor:'#7499b8', label: 'Feels like temperature', data: [] }
        ]
      },
      rainSnowDay:{
        labels: [],
        datasets: [
          { backgroundColor:'rgb(133, 188, 241)', label: 'Chance of rain as percentage', data: [] },
          { backgroundColor:'#7499b8', label: 'Chance of snow as percentage', data: [] }
        ]
      },
      cloudDay: {
        labels: [],
        datasets: [
          { backgroundColor:'rgb(133, 188, 241)', label: 'Cloudy', data: [] }
        ]
      },
      humidityDay: {
        labels: [],
        datasets: [
          { backgroundColor:'rgb(133, 188, 241)', label: 'Humidity', data: [] }
        ]
      },
      chartOptions: {
        responsive: true,
        plugins: {legend: {labels: {font: {size: 14}}}}
      },
      Weather:{
        city:'',
        current:{
          temp:'',
          wind:'',
          feel:'',
          humidity:'',
          twentyFourHours:[],
          sunMoon:{},
          svg:{},
        },
        days:[]
      },
      date:new Date(),
      SvgsDay:[
        {
          name:'sunny',
          svg: sunny,
          desk: 'Солнечно',
          background: 'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, #fff3d0 44.9%, #eaf7fc 96.1%)',
        },
        {
          name: 'partly cloudy',
          svg: partlyCloudy,
          desk: 'Переменная облачность',
          background: 'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, rgb(200, 223, 244) 44.9%, rgb(234, 247, 252) 96.1%)'
        },
        {
          name: 'variable rain and snow',
          svg: partlyRainingAndSnowing,
          desk: 'Переменный дождь и снег',
          background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, #ffe971 96.1%)'
        },
        {
          name: 'variable rain',
          svg: partlyRaining,
          desk: 'Переменный дождь',
          background: 'linear-gradient(179.1deg, rgb(254, 213, 102) -1.9%, rgb(200 223 244) 44.9%, rgb(234, 247, 252) 96.1%)'
        },
        {
          name: 'variable snow',
          svg: partlySnowing,
          desk: 'Переменный снег',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #f3ece3 96.1%)'
        },
        {
          name: 'drizzling rain',
          svg: drizzlingRain,
          desk: 'Моросящий дождь',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'rain',
          svg: rain,
          desk: 'Дождь',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'cloudy',
          svg: cloudy,
          desk: 'Облачно',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #bedfec 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'drizzling snow',
          svg: drizzlingSnow,
          desk: 'Моросящий снег',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'snowy',
          svg: snowy,
          desk: 'Снежно',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'drizzling rain and snow',
          svg: drizzlingRainAndSnow,
          desk: 'Моросящий дождь и снег',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'rain and snow',
          svg: rainAndSnow,
          desk: 'Дождь и снег',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'storm',
          svg: storm,
          desk: 'Гроза',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'sunny and storm',
          svg: sunnyStorm,
          desk: 'Гроза и солнце',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #dedbd1 96.1%)'
        },
        {
          name:'snow and storm',
          svg: snowStorm,
          desk: 'Гроза и снег',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'sunny, snow and storm',
          svg: sunnySnowStorm,
          desk: 'Гроза, солнце и снег',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'overcast',
          svg: overcast,
          desk: 'Пасмурно',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'blizzard',
          svg: fog,
          desk: 'Метель',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
      ],
      SvgsNight:[
        {
          name: 'moon',
          svg: moon,
          desk: 'Лунно',
          background: 'linear-gradient(179.1deg, rgb(178, 188, 197) -1.9%, rgb(176 186 189) 44.9%, rgb(249, 250, 250) 96.1%)',
        },
        {
          name: 'partly cloudy',
          svg: partlyCloudyMoon,
          desk: 'Переменная облачность',
          background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, rgb(178, 188, 197) 96.1%)'
        },
        {
          name: 'variable rain and snow',
          svg: partlyRainingAndSnowingMoon,
          desk: 'Переменный дождь и снег',
          background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, rgb(178, 188, 197) 96.1%)'
        },
        {
          name: 'variable rain',
          svg: partlyRainingMoon,
          desk: 'Переменный дождь',
          background: 'linear-gradient(179.1deg, rgb(178, 188, 197) -1.9%, rgb(200 223 244) 44.9%, rgb(234, 247, 252) 96.1%)'
        },
        {
          name: 'variable snow',
          svg: partlySnowingMoon,
          desk: 'Переменный снег',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, rgb(178, 188, 197) 96.1%)'
        },
        {
          name: 'drizzling rain',
          svg: drizzlingRain,
          desk: 'Моросящий дождь',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'rain',
          svg: rain,
          desk: 'Дождь',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'cloudy',
          svg: cloudy,
          desk: 'Облачно',
          background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #bedfec 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'drizzling snow',
          svg: drizzlingSnow,
          desk: 'Моросящий снег',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'snowy',
          svg: snowy,
          desk: 'Снежно',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'drizzling rain and snow',
          svg: drizzlingRainAndSnow,
          desk: 'Моросящий дождь и снег',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'rain and snow',
          svg: rainAndSnow,
          desk: 'Дождь и снег',
          background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)'
        },
        {
          name: 'storm',
          svg: storm,
          desk: 'Гроза',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'moon and storm',
          svg: moonStorm,
          desk: 'Гроза и луна',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #b6b6b6 96.1%)'
        },
        {
          name: 'snow and storm',
          svg: snowStorm,
          desk: 'Гроза и снег',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #bcbcbc 96.1%)'
        },
        {
          name: 'moon, snow and storm',
          svg: moonSnowStorm,
          desk: 'Гроза, луна и снег',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'overcast',
          svg: overcast,
          desk: 'Пасмурно',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name: 'blizzard',
          svg: fog,
          desk: 'Метель',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
      ],
      IncomingPictures:[
        {codes:[1000]},
        {codes:[1003]},
        {codes:[1069,1252]},
        {codes:[1063, 1192, 1243, 1246]},
        {codes:[1066, 1255, 1258, 1261, 1264]},
        {codes:[1150, 1153, 1180, 1183, 1186, 1189, 1240]},
        {codes:[1072, 1171, 1195]},
        {codes:[1006]},
        {codes:[1114, 1117, 1168, 1204,1210, 1213, 1216, 1219]},
        {codes:[1207, 1222, 1225]},
        {codes:[1198, 1237]},
        {codes:[1201, 1249]},
        {codes:[1276]},
        {codes:[1087, 1273]},
        {codes:[1282]},
        {codes:[1279]},
        {codes:[1009]},
        {codes:[1030, 1135, 1147]},
      ]
    }
  },
  methods:{
    async recalculateTheWeather(){
      let response  = await this.getWeather();
      if(response.status !== 200){
          this.show = false;
      }
      this.loading = false;
      this.getCurrentData(response.data);
      this.getWeekData(response.data);
    },
    async getWeather(){
      const API_KEY = "b77f6d044b784ff99b1160139232111"
      let q = this.selected;
      let days = '3'
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
      let { current, forecast } = data;
      let currentForecast = forecast.forecastday[0]

      this.Weather = {
        city: data.location.name,
        current: {
          temp: current.temp_c,
          wind: current.wind_kph,
          vis_km: current.vis_km,
          feel:current.feelslike_c,
          humidity: current.humidity,
          svg: this.putInSvg(this.Weather.current.is_day,current.condition.code)[0],
          name: this.putInSvg(this.Weather.current.is_day,current.condition.code)[1],
          background: this.putInSvg(this.Weather.current.is_day,current.condition.code)[2],
          precip_mm: current.precip_mm,
          wind_dir: current.wind_dir,
          wind_degree: current.wind_degree,
          cloudy: current.cloud,
          sunMoon: {
            start: currentForecast.astro.sunrise.slice(0,-3),
            end: Number(currentForecast.astro.sunset.slice(0,2))+12+':'+ currentForecast.astro.sunset.slice(3,5),
            moon_phase: currentForecast.astro.moon_phase.toLowerCase(),
          },
          twentyFourHours: []
        }
      }

      let array = [];

      this.tempDay = {
        labels: [],
        datasets: [
          {backgroundColor:'rgb(133, 188, 241)', label: 'Temperature', data: []},
          {backgroundColor:'#7499b8', label: 'Feels like temperature', data: []}
        ]
      };
      this.rainSnowDay={
        labels: [],
        datasets: [
          { backgroundColor:'rgb(133, 188, 241)', label: 'Chance of rain as percentage', data: [] },
          { backgroundColor:'#7499b8', label: 'Chance of snow as percentage', data: [] }
        ]
      };
      this.cloudDay= {
        labels: [],
        datasets: [
          { backgroundColor:'rgb(133, 188, 241)', label: 'Cloudy', data: [] }
        ]
      };
      this.humidityDay= {
        labels: [],
        datasets: [
          { backgroundColor:'rgb(133, 188, 241)', label: 'Humidity', data: [] }
        ]
      };

      for(let i=0; i < currentForecast.hour.length; i++){
        let timeDay = {};
        let code = currentForecast.hour[i].condition.code;

        if(i === 6 || i === 12){
          timeDay = {
            temp: currentForecast.hour[i].temp_c,
            name:i === 6 ? 'Morning' : 'Afternoon',
            svg: this.putInSvg(0,code)[0],
            svgName: this.putInSvg(0,code)[1],
          }
          this.Weather.current.twentyFourHours.push(timeDay);
        }
        if(i === 18){
          timeDay = {
            temp: currentForecast.hour[i].temp_c,
            name: 'Evening',
            svg: this.putInSvg(1,code)[0],
            svgName: this.putInSvg(1,code)[1],
          }
          this.Weather.current.twentyFourHours.push(timeDay);
        }

        array.push(currentForecast.hour[i].condition.code);
        let time = (currentForecast.hour[i].time).substring(10).trim();

        this.tempDay.labels.push(time);
        let temp_c = Math.round(currentForecast.hour[i].temp_c);
        this.tempDay.datasets[0].data.push(temp_c);
        let feelslike_c = Math.round(currentForecast.hour[i].feelslike_c);
        this.tempDay.datasets[1].data.push(feelslike_c);

        this.rainSnowDay.labels.push(time);
        let chance_of_rain = Math.round(currentForecast.hour[i].chance_of_rain);
        this.rainSnowDay.datasets[0].data.push(chance_of_rain);
        let chance_of_snow = Math.round(currentForecast.hour[i].chance_of_snow);
        this.rainSnowDay.datasets[1].data.push(chance_of_snow);

        this.cloudDay.labels.push(time);
        let cloud = currentForecast.hour[i].cloud;
        this.cloudDay.datasets[0].data.push(cloud);

        this.humidityDay.labels.push(time);
        let humidity = currentForecast.hour[i].humidity;
        this.humidityDay.datasets[0].data.push(humidity);
        
      }
      let codeSvg = array.sort((a,b) => array.filter(v => v===a).length - array.filter(v => v===b).length).pop();
      this.Weather.current.sunMoon.sun_phase = this.putInSvg(this.Weather.current.is_day,codeSvg)[1];
    },
    putInSvg(is_day, codePicture){
      for(const i in this.IncomingPictures) {
        for(const code of this.IncomingPictures[i].codes) {
          if(code === codePicture){
            if(is_day === 0){
              return [this.SvgsDay[i].svg, this.SvgsDay[i].name, this.SvgsDay[i].background ];
            }
            return [this.SvgsNight[i].svg, this.SvgsNight[i].name, this.SvgsNight[i].background ];
          }
        }
      }
    },
    getWeekData(data){
      let forecast = data.forecast;
      this.Weather.days = [];
      let forecastdays = forecast.forecastday;
      for(let jj=1; jj < forecastdays.length; jj++){
        let code = forecastdays[jj].day.condition.code;
        let day = {
          maxTemp: forecastdays[jj].day.maxtemp_c,
          minTemp: forecastdays[jj].day.mintemp_c,
          svg: this.putInSvg(0,code)[0],
          name: this.putInSvg(0,code)[1],
          number: new Date(this.getDayMs(jj)).toLocaleDateString(),
        };
        this.Weather.days.push(day);
      }
    },
  },
  async created() {
    await this.recalculateTheWeather()
  },


}
</script>

<style lang="scss">
  .mainSvg{
    width: 120%;
    margin-top: -10%;
    margin-left: -10%;
  }
  h4{
    font-size: 1.2rem !important;
  }
  .day_name{
    text-align: left;
    margin-bottom: 0 !important;
  }
  .sunMoonSvg{
    width: 255%;
    margin: 30% auto 0 -45%;
    display: block;
  }
  .svgDayOfTheWeek{
    width: 185%;
    margin: 30% auto 0 -15%;
    display: block;
  }
  .twentyFourHoursSvg{
    width: 125%;
    margin: -20% auto 0 -10%;
  }
  .twentyFourHoursTemp{
    font-size: 1.6rem;
    text-align: center;
    margin-top: -10%;
  }
  #app {
    background: linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%);
    min-height: 100vh;
    padding: 0;
    font-family: 'Roboto Condensed', sans-serif;
  }
  .container{
    padding: 0 !important;
  }
  .container_error{
    padding: 25% 0 0 !important;
  }
  .error{
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    h1{
      color: #db4b32;
      margin-left: 55px;
    }
    h1,h2{
      text-align: left;
    }
  }
  canvas{
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    margin: 0 0 2% 0;
  }
  .today{
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    text-align: left;
    color: black;
    padding: 15px;
    margin-bottom: 7px;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  .feels, .hours{
    text-align: left;
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    margin-bottom: 3%;
    padding: 15px;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  .today_time p, .today_time{
    color: black;
    text-align: left;
    margin-bottom: 0;
  }
  hr{
    margin-top: 0rem !important;
    margin-bottom: 0rem !important;
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
    font-size: 2rem;
    line-height: 2rem;
    text-align: center;
    margin-top: -8%;
    margin-bottom: 0;
  }
  select{
    background: transparent !important;
    border: none !important;
  }
  select:focus {
    outline: none;
  }
  option{
    background-color: rgba(255, 255, 255, 0.66) !important;
  }
  .city{
    font-size: 1.5rem;
    line-height: 2rem;
    font-weight: 500;
    text-align: center;
    width: 45%;
    margin: 0 auto 3%;
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 6px 12px 6px;
  }
  .line{
    height: 125px;
  }
  .conditions{
    font-size: 0.85rem !important;
    line-height: 1rem;
    text-align: center;
    margin-bottom: 0 !important;
  }
  .light{
    margin-top: 15%;
    font-size: 1.5rem;
    margin-bottom: 0 !important;
  }
  .sunMoon{
    margin-top: 25%;
  }
  .day{
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    h2{
      font-size: 1rem !important;
      font-weight: 500;
      margin-bottom: 0 !important;
    }
    .today_time p, .today_time{
      font-size: .9rem
    }
  }
  .col-3{
    padding: 0 !important;
  }
  .col, .col-3 {
    .flex-container > .day{
      width: 47%;
      margin: 0 1% 7px;
    }
  }
  .col{
    padding: 0 3%;
    .flex-container > .Line{
      width: 47.8%;
      margin: 0 8px 7px;
    }
  }
  .flex-container > .day{
    width: 15.16%;
    height: 125px;
    margin: 0 0.75% 1%;
  }
  .flex-container > .dd{
    width: 15.16%;
    margin: 0 0.75% 1%;
  }
  .flex-container, .hours{
    display: flex;
    flex-wrap: wrap;
  }
  .flex-container > p{
    width: 100%;
    margin: 0;
  }
  .hours > div{
    width: 31%;
    margin: 0 1%;
  }
  .addRow .col:first-child{
    padding-right: 0 !important;
  }
  .addRow .col:last-child{
    padding-left: 0 !important;
  }

  @media screen and (min-width: 992px) and (max-width: 1200px){
    .city{
      width: 40%;
      font-size: 1.2rem;
    }
    .col .flex-container > .Line {
      width: 47.5%;
    }
    .light {
      font-size: 1.2rem;
      margin-top: 25%;
    }
    .day h2 {
      font-size: .9rem !important;
    }
    .svgDayOfTheWeek {
      margin: 50% auto 0 -15%;
    }
    .sunMoonSvg {
      margin: 70% auto 0 -45%;
    }
  }
  @media screen and (min-width: 768px) and (max-width: 992px){
    .city{
      width: 55%;
      font-size: 1.2rem;
    }
    .mainRow{
      flex-direction: column;
      .col-3{
        display: flex;
        max-width: 100%;
        padding: 0 15px !important;
        & > div{
          width: 32%;
          margin: 0 0.75% 1%;
        }
      }
    }
    .col .flex-container > .Line {
      width: 47.4%;
    }
    .light {
      font-size: 1.2rem;
      margin-top: 25%;
    }
    .day h2 {
      font-size: .9rem !important;
    }
    .svgDayOfTheWeek {
      margin: 50% auto 0 -15%;
    }
    .sunMoonSvg {
      margin: 70% auto 0 -45%;
    }
  }
  @media screen and (min-width: 500px) and (max-width: 768px){
    .city{
      width: 93%;
      font-size: 1.2rem;
    }
    .row{
      margin: 0 !important;
    }
    .mainRow, .addRow{
      flex-direction: column;
      .col{
        padding: 0 5px !important;
      }
      .col-3{
        max-width: 100%;
        padding: 0 15px !important;
      }
    }
    .mainSvg {
      width: 60%;
      float: left;
      display: block;
      margin: 10% auto 0;
    }
    .temp{
      margin-top: 15%;
    }
    .twentyFourHoursSvg {
      width: 65%;
      margin: -10% auto 5% -10%;
    }
    .col .flex-container > .day, .col-3 .flex-container > .day {
      width: 48%;
      margin: 0 1% 7px;
    }
    .col .flex-container > .Line {
      width: 100%;
    }
    .light {
      font-size: 1.2rem;
      margin-top: 15%;
    }
    .day h2 {
      font-size: .9rem !important;
    }
    .svgDayOfTheWeek {
      width: 145%;
      margin: 15% auto 0 auto;
    }
    .sunMoonSvg {
      margin: 25% auto 0 auto;
      width: 155%;
    }
    .sunMoon {
      margin-top: 20%;
    }

  }
  @media screen and (max-width: 500px){
    .city{
      width: 93%;
      font-size: 1.2rem;
    }
    .row{
      margin: 0 !important;
    }
    .addRow{
      padding: 0 5px;
    }
    .mainRow, .addRow, .darkLight{
      flex-direction: column;
      .col{
        padding: 0 5px !important;
      }
      .col-3,.col-4, .col-5{
        max-width: 100%;
        padding: 0 15px !important;
      }
    }
    .mainSvg {
      width: 80%;
      display: block;
      margin: -10% auto 0;
    }
    .temp{
      margin-top: 0;
    }
    .twentyFourHoursSvg {
      width: 65%;
      margin:  -10% auto 5% 20%;
    }
    .col .flex-container > .day, .col-3 .flex-container > .day {
      width: 48%;
      margin: 0 1% 7px;
    }
    .col .flex-container > .Line {
      width: 99%;
      padding: 0 10px 7px;
      margin: 0;
    }
    .light {
      font-size: 1.2rem;
      margin-top: -4%;
      margin-left: 35%;
    }
    .additionally_h{
      padding-left: 5px;
    }
    .day h2 {
      font-size: .85rem !important;
    }
    .sunMoonSvg, .svgDayOfTheWeek {
      margin: 5% auto 0 auto;
      width: 65%;
    }
    .sunMoon {
      margin-top: 0;
    }

  }
</style>
