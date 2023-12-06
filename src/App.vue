<template>
  <div id="app" :style="{background:Weather.current.background}">
    <b-container v-if="!loading">
        <h1>Weather Forecast</h1>
        <b-row>
          <b-col cols="3">
<!--            <div class="city">{{Weather.city}}</div>-->
            <h4>Today:</h4>
            <div class="today line1">
              <b-row>
                <b-col cols="5">
                  <component  :is="Weather.current.svg" class="mainSvg"/>
                  <div class="temp">{{Math.round(Weather.current.temp)}}°</div>
                  <p class="conditions"><b>{{Weather.current.name}}</b></p>
                </b-col>
                <b-col>
                  <p class="today_time"><b>{{date.toLocaleString('en-US', { weekday: 'long' })}}</b></p>
                  <hr/>
                  <p class="today_time">{{date.toLocaleDateString()}}</p>
                  <p class="today_time">{{date.toLocaleTimeString().slice(0,-3)}}</p>
                  <hr/>
                  <p class="today_time"> Feels like: {{Math.round(Weather.current.feel)}}°</p>
                </b-col>
              </b-row>
            </div>
            <h4>Detailed description:</h4>
            <div class="feels">
              <div class="number flex-container">
                <div>Cloudy:<b>{{Weather.current.cloudy}}%</b></div>
                <div>Humidity: <b>{{Weather.current.humidity}}%</b></div>
                <div>Precipitation: <b>{{Weather.current.precip_mm}} mm</b></div>
                <div>Wind: <b>{{Weather.current.wind}} km/h</b></div>
                <div>Visibility:<b>{{Weather.current.vis_km}} km</b></div>
                <div>Wind direction by compass: <b>{{Weather.current.wind_dir}}</b></div>
                <div>Wind direction in degree: <b>{{Weather.current.wind_degree}}</b></div>
              </div>
            </div>
            <div>
              <h4>Twenty-four hours:</h4>
              <div class="feels">
                <b-row>
                  <b-col v-for="(day, index) in Weather.current.twentyFourHours" :key="index">
                    <p><b>{{day.name}}</b></p>
                    <component :key="index" :is="day.svg" class="twentyFourHoursSvg"/>
                    <div class="twentyFourHoursTemp">{{Math.round(day.temp)}}°</div>
                    <p class="conditions"><b>{{day.svgName}}</b></p>
                  </b-col>
                </b-row>
              </div>
            </div>
          </b-col>
          <b-col>
            <div class="flex-container">
              <div>
                <h4>Light and dark time of day:</h4>
                  <div class="flex-container">
                    <div class="day">
                      <p class="day_name">Light time of day:</p>
                      <hr/>
                      <b-row>
                        <b-col cols="4">
                          <component :is="SvgsDay[0].svg" class="sunMoonSvg"/>
                        </b-col>
                        <b-col>
                          <p class="light conditions"><b> from {{Weather.current.sunMoon.start}} to {{Weather.current.sunMoon.end}}</b></p>
                          <p class="conditions"><b>{{Weather.current.sunMoon.sun_phase}}</b></p>
                        </b-col>
                      </b-row>
                    </div>
                    <div class="day">
                      <p class="day_name">Dark time of day:</p>
                      <hr/>
                      <b-row>
                        <b-col cols="4">
                          <component :is="SvgsNight[0].svg" class="sunMoonSvg"/>
                        </b-col>
                        <b-col>
                          <p class="light conditions"><b> from {{Weather.current.sunMoon.end}} to {{Weather.current.sunMoon.start}}</b></p>
                          <p class="conditions"><b>{{Weather.current.sunMoon.moon_phase}}</b></p>
                        </b-col>
                      </b-row>
                    </div>
                </div>
              </div>
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
                    <b-row>
                      <b-col cols="5">
                        <component :key="index" :is="day.svg" width="100%" class="svgDayOfTheWeek"/>
                      </b-col>
                      <b-col>
                        <div class="light">{{Math.round(day.minTemp)}}°   {{Math.round(day.maxTemp)}}°</div>
                        <p class="conditions"><b>{{day.name}}</b></p>
                      </b-col>
                    </b-row>
                  </div>
                </div>
              </div>
            </div>
            <h4>Hourly forecast</h4>
            <div class="flex-container">
              <div><WeatherChart :chartOptions="chartOptions" :chartData="tempDay" type="Line"/></div>
              <div><WeatherChart :chartOptions="chartOptions" :chartData="rainSnowDay" type="Line"/></div>
              <div><WeatherChart :chartOptions="chartOptions" :chartData="cloudDay" type="Line"/></div>
              <div><WeatherChart :chartOptions="chartOptions" :chartData="humidityDay" type="Line"/></div>
            </div>
          </b-col>
        </b-row>
      </b-container>
  </div>
</template>

<script>
  import axios from 'axios'

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
      loading: true,
      tempDay: {
        labels: [],
        datasets: [
          { label: 'Average temperature', data: [] },
          { label: 'Feels like temperature', data: [] }
        ]
      },
      cloudHumidityDay:{
        labels: [],
        datasets: [
          { label: 'Cloudy', data: [] },
          { label: 'Humidity', data: [] }
        ]
      },
      rainSnowDay:{
        labels: [],
        datasets: [
          { label: 'Chance of rain as percentage', data: [] },
          { label: 'Chance of snow as percentage', data: [] }
        ]
      },
      cloudDay: {
        labels: [],
        datasets: [
          { label: 'Cloudy', data: [] }
        ]
      },
      humidityDay: {
        labels: [],
        datasets: [
          { label: 'Humidity', data: [] }
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
        {
          name:'blizzard',
          svg:fog,
          desk:'Метель',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
      ],
      SvgsNight:[
        {
          name:'sunny',
          svg:moon,
          desk:'Солнечно',
          background:'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, #fff3d0 44.9%, #eaf7fc 96.1%)',
        },
        {
          name:'partly cloudy',
          svg: partlyCloudyMoon,
          desk:'Переменная облачность',
          background: 'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, rgb(200, 223, 244) 44.9%, rgb(234, 247, 252) 96.1%)'
        },
        {
          name:'variable rain and snow',
          svg: partlyRainingAndSnowingMoon,
          desk:'Переменный дождь и снег',
          background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, #ffe971 96.1%)'
        },
        {
          name:'variable rain',
          svg: partlyRainingMoon,
          desk:'Переменный дождь',
          background: 'linear-gradient(179.1deg, rgb(254, 213, 102) -1.9%, rgb(200 223 244) 44.9%, rgb(234, 247, 252) 96.1%)'
        },
        {
          name:'variable snow',
          svg: partlySnowingMoon,
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
          svg:moonStorm,
          desk:'Гроза и луна',
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
          svg:moonSnowStorm,
          desk:'Гроза, луна и снег',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'overcast',
          svg:overcast,
          desk:'Пасмурно',
          background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)'
        },
        {
          name:'blizzard',
          svg:fog,
          desk:'Метель',
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
          codes:[1009]
        },
        {
          codes:[1030, 1135, 1147]
        },
      ]
    }
  },
  methods:{
    async getWeather(){
      const API_KEY = "b77f6d044b784ff99b1160139232111"
      let q = 'Moscow';
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
      let current =  data.current;
      let forecast = data.forecast;
      this.Weather.city = data.location.name;
      this.Weather.current.temp = current.temp_c;
      this.Weather.current.wind = current.wind_kph;
      this.Weather.current.vis_km = current.vis_km;
      this.Weather.current.feel = current.feelslike_c;
      this.Weather.current.humidity = current.humidity;
      this.Weather.current.svg = this.putInSvg(this.Weather.current.is_day,current.condition.code)[0];
      this.Weather.current.name = this.putInSvg(this.Weather.current.is_day,current.condition.code)[1];
      this.Weather.current.background = this.putInSvg(this.Weather.current.is_day,current.condition.code)[2];
      this.Weather.current.precip_mm = current.precip_mm;
      this.Weather.current.wind_dir = current.wind_dir;
      this.Weather.current.wind_degree = current.wind_degree;
      this.Weather.current.cloudy = current.cloud;


      for(let i=6; i < forecast.forecastday[0].hour.length; i=i+6){
        let time = {};

        time.temp = forecast.forecastday[0].hour[i].temp_c;
        let times = forecast.forecastday[0].hour[i].time.substring(10).trim();

        let code = forecast.forecastday[0].hour[i].condition.code;

        if(times === '06:00' && i === 6){
          time.name = 'Morning';
          time.svg = this.putInSvg(0,code)[0];
          time.svgName = this.putInSvg(0,code)[1];
        }
        if(times === '12:00' && i === 12){
          time.name = 'Afternoon';
          time.svg = this.putInSvg(0,code)[0];
          time.svgName = this.putInSvg(0,code)[1];
        }
        if(times === '18:00' && i === 18){
          time.name = 'Evening';
          time.svg = this.putInSvg(1,code)[0];
          time.svgName = this.putInSvg(1,code)[1];
        }

        this.Weather.current.twentyFourHours.push(time);
      }

      this.Weather.current.sunMoon.start = forecast.forecastday[0].astro.sunrise.slice(0,-3);
      this.Weather.current.sunMoon.end = Number(forecast.forecastday[0].astro.sunset.slice(0,2))+12+':'+ forecast.forecastday[0].astro.sunset.slice(3,5);
      this.Weather.current.sunMoon.moon_phase = forecast.forecastday[0].astro.moon_phase.toLowerCase();
      let array = []
      for(let i=0; i < forecast.forecastday[0].hour.length; i++){
        array.push(forecast.forecastday[0].hour[i].condition.code);
        let time = (forecast.forecastday[0].hour[i].time).substring(10).trim();
        let temp_c = Math.round(forecast.forecastday[0].hour[i].temp_c);
        let feelslike_c = Math.round(forecast.forecastday[0].hour[i].feelslike_c);
        let chance_of_rain = Math.round(forecast.forecastday[0].hour[i].chance_of_rain);
        let chance_of_snow = Math.round(forecast.forecastday[0].hour[i].chance_of_snow);
        let cloud = forecast.forecastday[0].hour[i].cloud;
        let humidity = forecast.forecastday[0].hour[i].humidity;
        

        this.tempDay.labels.push(time);
        this.tempDay.datasets[0].backgroundColor = 'rgb(133, 188, 241)';
        this.tempDay.datasets[0].data.push(temp_c);
        this.tempDay.datasets[1].backgroundColor = '#7499b8';
        this.tempDay.datasets[1].data.push(feelslike_c);

        this.cloudHumidityDay.labels.push(time);
        this.cloudHumidityDay.datasets[0].backgroundColor = 'rgb(133, 188, 241)';
        this.cloudHumidityDay.datasets[0].data.push(cloud);
        this.cloudHumidityDay.datasets[1].backgroundColor = '#7499b8';
        this.cloudHumidityDay.datasets[1].data.push(humidity);

        this.rainSnowDay.labels.push(time);
        this.rainSnowDay.datasets[0].backgroundColor = 'rgb(133, 188, 241)';
        this.rainSnowDay.datasets[0].data.push(chance_of_rain);
        this.rainSnowDay.datasets[1].backgroundColor = '#7499b8';
        this.rainSnowDay.datasets[1].data.push(chance_of_snow);

        this.cloudDay.labels.push(time);
        this.cloudDay.datasets[0].backgroundColor = 'rgb(133, 188, 241)'
        this.cloudDay.datasets[0].data.push(cloud);

        this.humidityDay.labels.push(time);
        this.humidityDay.datasets[0].backgroundColor = 'rgb(133, 188, 241)'
        this.humidityDay.datasets[0].data.push(humidity);
        
      }
      let codeSvg = array.sort((a,b) => array.filter(v => v===a).length - array.filter(v => v===b).length).pop();
      this.Weather.current.sunMoon.sun_phase = this.putInSvg(this.Weather.current.is_day,codeSvg)[1]
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
      for(let jj=1; jj < forecast.forecastday.length; jj++){
        let day = {};
        day.maxTemp = forecast.forecastday[jj].day.maxtemp_c;
        day.minTemp = forecast.forecastday[jj].day.mintemp_c;
        let code = forecast.forecastday[jj].day.condition.code;
        day.svg = this.putInSvg(0,code)[0];
        day.name = this.putInSvg(0,code)[1];
        day.number = new Date(this.getDayMs(jj)).toLocaleDateString();
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
    console.log('this.Weather', this.Weather);
  },


}
</script>

<style lang="scss">
  h1{
    font-size: 2rem !important;
    margin-bottom: .5rem !important;
  }
  h4{
    font-size: 1.2rem !important;
  }
  .day_name{
    text-align: left;
    margin-bottom: 0 !important;
  }
  .sunMoonSvg{
    width: 285%;
    margin: 45% auto 0 -45%;
    display: block;
  }
  .svgDayOfTheWeek{
    width: 185%;
    margin: 45% auto 0 -15%;
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
  h1{
    margin-bottom: 1rem !important;
    text-shadow: 1.5px 1.5px 1.5px #f3e0e0;
  }
  #app {
    background: linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%);
    min-height: 100vh;
    text-align: center;
    padding: 20px 0;
    font-family: 'Roboto Condensed', sans-serif;
  }
  .container{
    padding: 0 !important;
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
    font-size: 1.4rem;
    text-align: left;
    color: black;
    padding: 15px;
    margin-bottom: 7px;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  .feels{
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    margin-bottom: 3%;
    padding: 15px;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  .today_time{
    color: black;
    text-align: left;
    font-size: 1.1rem;
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
    margin-bottom: 5%;
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
  .number{
    font-size: 1.03rem;
    text-align: left;
    color: black;
  }
  .line1{
    height: 125px;
  }
  .light, .dark{
    height: 35px;
    font-size: 1.6rem;
    text-align: center;
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
    .today_time{
      font-size: .9rem
    }
  }
  .col-3{
    padding: 0 !important;
  }
  .col, .col-3 {
    .flex-container > .day{
      width: 47%;
      margin: 0 1% 0;
    }
  }
  .col{
    .flex-container > div{
      width: 48%;
      margin: 0 7px 7px;
    }
  }
  .flex-container > .day{
    width: 15.16%;
    height: 125px;
    //height: 29vh;
    margin: 0 0.75% 1%;
  }
  .flex-container > .dd{
    width: 15.16%;
    margin: 0 0.75% 1%;
  }
  .flex-container{
    display: flex;
    flex-wrap: wrap;
  }
  .feels .flex-container > div{
    width: 100%;
    margin: 0;
  }
  .col{
    padding: 0 3%;
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
