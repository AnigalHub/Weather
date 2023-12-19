<template>
  <div id="app" :style="{ background: Weather.current.background }">
    <b-container v-if="loaded">
      <div class="city">
        <span>Choose one of the options: </span>
        <select v-model="selected" @click="recalculateTheWeather()">
          <option v-for="(option, index) in options" :key="index">
            {{ option }}
          </option>
        </select>
      </div>
      <b-row class="mainRow">
        <b-col cols="3">
          <div>
            <h4>Today:</h4>
            <div class="today">
              <b-row>
                <b-col cols="5">
                  <component :is="Weather.current.svg" class="mainSvg" />
                  <p class="temp">{{ Math.round(Weather.current.temp) }}°</p>
                  <p class="conditions">
                    <b>{{ Weather.current.name }}</b>
                  </p>
                </b-col>
                <b-col class="today_inf">
                  <p><b>{{ date.toLocaleString('en-US', { weekday: 'long' }) }}</b></p>
                  <hr/>
                  <p>{{ date.toLocaleDateString() }}</p>
                  <p>{{ date.toLocaleTimeString().slice(0, -3) }}</p>
                  <hr/>
                  <p>Feels like: {{ Math.round(Weather.current.feel) }}°</p>
                </b-col>
              </b-row>
            </div>
          </div>
          <div>
            <h4>Detailed description:</h4>
            <div class="feels">
              <div class="flex-container">
                <p>Cloudy: <b>{{ Weather.current.cloudy }}%</b></p>
                <p>Humidity: <b>{{ Weather.current.humidity }}%</b></p>
                <p>Precipitation: <b>{{ Weather.current.precip_mm }} mm</b></p>
                <p>Wind: <b>{{ Weather.current.wind }} km/h</b></p>
                <p>Visibility:<b>{{ Weather.current.vis_km }} km</b></p>
                <p>Wind direction by compass:<b>{{ Weather.current.wind_dir }}</b></p>
                <p>Wind direction in degree:<b>{{ Weather.current.wind_degree }}</b></p>
              </div>
            </div>
          </div>
          <div>
            <h4>Twenty-four hours:</h4>
            <div class="hours">
              <div v-for="(day, index) in Weather.current.twentyFourHours" :key="index">
                <p class="text-center"><b>{{ day.name }}</b></p>
                <component :is="day.svg" :key="index" class="twentyFourHoursSvg"/>
                <div class="twentyFourHoursTemp">{{ Math.round(day.temp) }}°</div>
                <p class="conditions"><b>{{ day.svgName }}</b></p>
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
                    <b-col cols="4"><component :is="SvgsDay[0].svg" class="sunMoonSvg" /></b-col>
                    <b-col>
                      <p class="sunMoon conditions">
                        <b>from {{ Weather.current.sunMoon.start }} to {{ Weather.current.sunMoon.end }}</b>
                      </p>
                      <p class="conditions"><b>{{ Weather.current.sunMoon.sun_phase }}</b></p>
                    </b-col>
                  </b-row>
                </div>
                <div class="day">
                  <p class="day_name">Dark time of day:</p>
                  <hr/>
                  <b-row class="darkLight">
                    <b-col cols="4"><component :is="SvgsNight[0].svg" class="sunMoonSvg" /></b-col>
                    <b-col>
                      <p class="sunMoon conditions"><b>from {{ Weather.current.sunMoon.end }} to {{ Weather.current.sunMoon.start }}</b></p>
                      <p class="conditions"><b>{{ Weather.current.sunMoon.moon_phase }}</b></p>
                    </b-col>
                  </b-row>
                </div>
              </div>
            </b-col>
            <b-col>
              <div>
                <h4>Coming days:</h4>
                <div class="flex-container">
                  <div v-for="(day, index) in Weather.days" :key="index" class="day">
                    <b-row>
                      <b-col>
                        <h2>{{ getDayOfTheWeek(index + 1) }}</h2>
                      </b-col>
                      <b-col>
                        <div class="today_inf">{{ day.number }}</div>
                      </b-col>
                    </b-row>
                    <hr/>
                    <b-row class="darkLight">
                      <b-col cols="5">
                        <component :is="day.svg" :key="index" width="100%" class="svgDayOfTheWeek"/>
                      </b-col>
                      <b-col>
                        <p class="darkLight_temp">
                          {{ Math.round(day.minTemp) }}°
                          {{ Math.round(day.maxTemp) }}°
                        </p>
                        <p class="conditions"><b>{{ day.name }}</b></p>
                      </b-col>
                    </b-row>
                  </div>
                </div>
              </div>
            </b-col>
          </b-row>
          <h4>Hourly forecast</h4>
          <div class="flex-container">
            <div class="chart"><weather-chart :chartOptions="chartOptions" :chartData="tempDay" type="Line"/></div>
            <div class="chart"><weather-chart :chartOptions="chartOptions" :chartData="rainSnowDay" type="Line"/></div>
            <div class="chart"><weather-chart :chartOptions="chartOptions" :chartData="cloudDay" type="Line"/></div>
            <div class="chart"><weather-chart :chartOptions="chartOptions" :chartData="humidityDay" type="Line"/></div>
          </div>
        </b-col>
      </b-row>
    </b-container>
    <b-container v-else class="container_error">
      <div class="error">
        <component :is="svgError" />
        <h1>Error</h1>
        <h2>An error has occurred. We apologize. We will fix it as soon as possible and are sincerely sorry for any inconvenience caused.</h2>
      </div>
    </b-container>
  </div>
</template>

<script>
  import axios from 'axios';

  import error from './components/icons/error';
  import sunny from './components/icons/sunny';
  import moon from './components/icons/moon';
  import partlyCloudy from '@/components/icons/partlyCloudy';
  import partlyCloudyMoon from '@/components/icons/partlyCloudyMoon';
  import partlyRainingAndSnowing from '@/components/icons/partlyRainingAndSnowing';
  import partlyRainingAndSnowingMoon from '@/components/icons/partlyRainingAndSnowingMoon';
  import partlyRaining from '@/components/icons/partlyRaining';
  import partlyRainingMoon from '@/components/icons/partlyRainingMoon';
  import partlySnowing from '@/components/icons/partlySnowing';
  import partlySnowingMoon from '@/components/icons/partlySnowingMoon';
  import drizzlingRain from './components/icons/drizzlingRain';
  import rain from './components/icons/rain';
  import cloudy from '@/components/icons/cloudy';
  import drizzlingSnow from '@/components/icons/drizzlingSnow';
  import snowy from '@/components/icons/snowy';
  import drizzlingRainAndSnow from '@/components/icons/drizzlingRainAndSnow';
  import rainAndSnow from '@/components/icons/rainAndSnow';
  import storm from '@/components/icons/storm';
  import sunnyStorm from '@/components/icons/sunnyStorm';
  import moonStorm from '@/components/icons/moonStorm';
  import snowStorm from '@/components/icons/snowStorm';
  import sunnySnowStorm from '@/components/icons/sunnySnowStorm';
  import moonSnowStorm from '@/components/icons/moonSnowStorm';
  import overcast from '@/components/icons/overcast';
  import fog from '@/components/icons/fog';
  import WeatherChart from '@/components/WeatherChart';

  export default {
    name: 'App',
    components: { WeatherChart },
    data() {
      return {
        svgError: error,
        loaded: false,
        selected: 'Moscow',
        options: ['Moscow', 'Kolomna', 'Ryazan', 'Ryazhsk', 'Vladimir'],
        tempDay: {
          labels: [],
          datasets: [
            {
              backgroundColor: 'rgb(133, 188, 241)',
              label: 'Temperature',
              data: [],
            },
            {
              backgroundColor: '#7499b8',
              label: 'Feels like temperature',
              data: [],
            },
          ],
        },
        rainSnowDay: {
          labels: [],
          datasets: [
            {
              backgroundColor: 'rgb(133, 188, 241)',
              label: 'Chance of rain as percentage',
              data: [],
            },
            {
              backgroundColor: '#7499b8',
              label: 'Chance of snow as percentage',
              data: [],
            },
          ],
        },
        cloudDay: {
          labels: [],
          datasets: [
            { backgroundColor: 'rgb(133, 188, 241)', label: 'Cloudy', data: [] },
          ],
        },
        humidityDay: {
          labels: [],
          datasets: [
            {
              backgroundColor: 'rgb(133, 188, 241)',
              label: 'Humidity',
              data: [],
            },
          ],
        },
        chartOptions: {
          responsive: true,
          plugins: { legend: { labels: { font: { size: 14 } } } },
        },
        Weather: {
          city: '',
          current: {
            temp: '',
            wind: '',
            feel: '',
            humidity: '',
            twentyFourHours: [],
            sunMoon: {},
            svg: {},
          },
          days: [],
        },
        date: new Date(),
        SvgsDay: [
          {
            name: 'sunny',
            svg: sunny,
            desk: 'Солнечно',
            background: 'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, #fff3d0 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'partly cloudy',
            svg: partlyCloudy,
            desk: 'Переменная облачность',
            background: 'linear-gradient(179.1deg, rgb(254 236 162) -1.9%, rgb(200, 223, 244) 44.9%, rgb(234, 247, 252) 96.1%)',
          },
          {
            name: 'variable rain and snow',
            svg: partlyRainingAndSnowing,
            desk: 'Переменный дождь и снег',
            background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, #ffe971 96.1%)',
          },
          {
            name: 'variable rain',
            svg: partlyRaining,
            desk: 'Переменный дождь',
            background: 'linear-gradient(179.1deg, rgb(254, 213, 102) -1.9%, rgb(200 223 244) 44.9%, rgb(234, 247, 252) 96.1%)',
          },
          {
            name: 'variable snow',
            svg: partlySnowing,
            desk: 'Переменный снег',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #f3ece3 96.1%)',
          },
          {
            name: 'drizzling rain',
            svg: drizzlingRain,
            desk: 'Моросящий дождь',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'rain',
            svg: rain,
            desk: 'Дождь',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'cloudy',
            svg: cloudy,
            desk: 'Облачно',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #bedfec 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'drizzling snow',
            svg: drizzlingSnow,
            desk: 'Моросящий снег',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'snowy',
            svg: snowy,
            desk: 'Снежно',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'drizzling rain and snow',
            svg: drizzlingRainAndSnow,
            desk: 'Моросящий дождь и снег',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'rain and snow',
            svg: rainAndSnow,
            desk: 'Дождь и снег',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'storm',
            svg: storm,
            desk: 'Гроза',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'sunny and storm',
            svg: sunnyStorm,
            desk: 'Гроза и солнце',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #dedbd1 96.1%)',
          },
          {
            name: 'snow and storm',
            svg: snowStorm,
            desk: 'Гроза и снег',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'sunny, snow and storm',
            svg: sunnySnowStorm,
            desk: 'Гроза, солнце и снег',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'overcast',
            svg: overcast,
            desk: 'Пасмурно',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'blizzard',
            svg: fog,
            desk: 'Метель',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
        ],
        SvgsNight: [
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
            background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, rgb(178, 188, 197) 96.1%)',
          },
          {
            name: 'variable rain and snow',
            svg: partlyRainingAndSnowingMoon,
            desk: 'Переменный дождь и снег',
            background: 'linear-gradient(179.1deg, #add2f5 -1.9%, #D5F3FF 44.9%, rgb(178, 188, 197) 96.1%)',
          },
          {
            name: 'variable rain',
            svg: partlyRainingMoon,
            desk: 'Переменный дождь',
            background: 'linear-gradient(179.1deg, rgb(178, 188, 197) -1.9%, rgb(200 223 244) 44.9%, rgb(234, 247, 252) 96.1%)',
          },
          {
            name: 'variable snow',
            svg: partlySnowingMoon,
            desk: 'Переменный снег',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, rgb(178, 188, 197) 96.1%)',
          },
          {
            name: 'drizzling rain',
            svg: drizzlingRain,
            desk: 'Моросящий дождь',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'rain',
            svg: rain,
            desk: 'Дождь',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #D5F3FF 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'cloudy',
            svg: cloudy,
            desk: 'Облачно',
            background: 'linear-gradient(179.1deg, #85BCF1 -1.9%, #bedfec 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'drizzling snow',
            svg: drizzlingSnow,
            desk: 'Моросящий снег',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'snowy',
            svg: snowy,
            desk: 'Снежно',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'drizzling rain and snow',
            svg: drizzlingRainAndSnow,
            desk: 'Моросящий дождь и снег',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'rain and snow',
            svg: rainAndSnow,
            desk: 'Дождь и снег',
            background: 'linear-gradient(179.1deg, #b2bcc5 -1.9%, #bedfec 44.9%, #f9fafa 96.1%)',
          },
          {
            name: 'storm',
            svg: storm,
            desk: 'Гроза',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'moon and storm',
            svg: moonStorm,
            desk: 'Гроза и луна',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #b6b6b6 96.1%)',
          },
          {
            name: 'snow and storm',
            svg: snowStorm,
            desk: 'Гроза и снег',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #bcbcbc 96.1%)',
          },
          {
            name: 'moon, snow and storm',
            svg: moonSnowStorm,
            desk: 'Гроза, луна и снег',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'overcast',
            svg: overcast,
            desk: 'Пасмурно',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
          {
            name: 'blizzard',
            svg: fog,
            desk: 'Метель',
            background: 'linear-gradient(179.1deg, #7096b7 -1.9%, #c5e8fc 44.9%, #eaf7fc 96.1%)',
          },
        ],
        IncomingPictures: [
          { codes: [1000] },
          { codes: [1003] },
          { codes: [1069, 1252] },
          { codes: [1063, 1192, 1243, 1246] },
          { codes: [1066, 1255, 1258, 1261, 1264] },
          { codes: [1150, 1153, 1180, 1183, 1186, 1189, 1240] },
          { codes: [1072, 1171, 1195] },
          { codes: [1006] },
          { codes: [1114, 1117, 1168, 1204, 1210, 1213, 1216, 1219] },
          { codes: [1207, 1222, 1225] },
          { codes: [1198, 1237] },
          { codes: [1201, 1249] },
          { codes: [1276] },
          { codes: [1087, 1273] },
          { codes: [1282] },
          { codes: [1279] },
          { codes: [1009] },
          { codes: [1030, 1135, 1147] },
        ],
      };
    },
    async created() {
      if(process.env.VUE_APP_API_KEY){
        await this.recalculateTheWeather();
      }
    },
    methods: {
      /** Подсчет погоды на все дни */
      async recalculateTheWeather() {
        const response = await this.getWeather();
        if (response.status !== 200) {
          this.loaded = true;
        }
        this.getCurrentData(response.data);
        this.getWeekData(response.data);
      },
      /** Получение погоды */
      async getWeather() {
        try{
          const API_KEY = process.env.VUE_APP_API_KEY;
          const q = this.selected;
          const days = '3';
          const response = await axios.get(
                  `http://api.weatherapi.com/v1/forecast.json`,
                  {
                    params: {
                      key: API_KEY,
                      q,
                      days,
                    },
                  },
          );
          return response;
        }
        catch (error) {
          console.log('Error:', error)
        }
      },
      /** Подсчет миллисекунд дня */
      getDayMs(i) {
        const date = new Date();
        return date.setDate(date.getDate() + i);
      },
      /** Подсчет даты */
      getDate(i) {
        const day = this.getDayMs(i);
        return new Date(day).toLocaleDateString();
      },
      /** Подсчет дня недели */
      getDayOfTheWeek(i) {
        const day = this.getDayMs(i);
        return new Date(day).toLocaleString('en-US', { weekday: 'long' });
      },
      /** Получение данных о погоде по текущему дню */
      getCurrentData(data) {
        const { current, forecast } = data;
        const currentForecast = forecast.forecastday[0];
        const [putSvg, putName, putBackground] = this.putInSvg(
                this.Weather.current.is_day,
                current.condition.code,
        );
        this.Weather = {
          city: data.location.name,
          current: {
            temp: current.temp_c,
            wind: current.wind_kph,
            vis_km: current.vis_km,
            feel: current.feelslike_c,
            humidity: current.humidity,
            svg: putSvg,
            name: putName,
            background: putBackground,
            precip_mm: current.precip_mm,
            wind_dir: current.wind_dir,
            wind_degree: current.wind_degree,
            cloudy: current.cloud,
            sunMoon: {
              start: currentForecast.astro.sunrise.slice(0, -3),
              end: `${ Number(currentForecast.astro.sunset.slice(0, 2)) + 12}:${currentForecast.astro.sunset.slice(3, 5)}`,
              moon_phase: currentForecast.astro.moon_phase.toLowerCase(),
            },
            twentyFourHours: [],
          },
        };

        const array = [];

        this.tempDay = {
          labels: [],
          datasets: [
            {
              backgroundColor: 'rgb(133, 188, 241)',
              label: 'Temperature',
              data: [],
            },
            {
              backgroundColor: '#7499b8',
              label: 'Feels like temperature',
              data: [],
            },
          ],
        };
        this.rainSnowDay = {
          labels: [],
          datasets: [
            {
              backgroundColor: 'rgb(133, 188, 241)',
              label: 'Chance of rain as percentage',
              data: [],
            },
            {
              backgroundColor: '#7499b8',
              label: 'Chance of snow as percentage',
              data: [],
            },
          ],
        };
        this.cloudDay = {
          labels: [],
          datasets: [
            { backgroundColor: 'rgb(133, 188, 241)', label: 'Cloudy', data: [] },
          ],
        };
        this.humidityDay = {
          labels: [],
          datasets: [
            {
              backgroundColor: 'rgb(133, 188, 241)',
              label: 'Humidity',
              data: [],
            },
          ],
        };

        for (let i = 0; i < currentForecast.hour.length; i++) {
          let timeDay = {};
          const { code } = currentForecast.hour[i].condition;

          if (i === 6 || i === 12) {
            const [putSvg, putName] = this.putInSvg(0, code);
            timeDay = {
              temp: currentForecast.hour[i].temp_c,
              name: i === 6 ? 'Morning' : 'Afternoon',
              svg: putSvg,
              svgName: putName,
            };
            this.Weather.current.twentyFourHours.push(timeDay);
          }

          if (i === 18) {
            const [putSvg, putName] = this.putInSvg(1, code);
            timeDay = {
              temp: currentForecast.hour[i].temp_c,
              name: 'Evening',
              svg: putSvg,
              svgName: putName,
            };
            this.Weather.current.twentyFourHours.push(timeDay);
          }

          array.push(currentForecast.hour[i].condition.code);
          const time = currentForecast.hour[i].time.substring(10).trim();

          this.tempDay.labels.push(time);
          const temp_c = Math.round(currentForecast.hour[i].temp_c);
          this.tempDay.datasets[0].data.push(temp_c);
          const feelslike_c = Math.round(currentForecast.hour[i].feelslike_c);
          this.tempDay.datasets[1].data.push(feelslike_c);

          this.rainSnowDay.labels.push(time);
          const chance_of_rain = Math.round(currentForecast.hour[i].chance_of_rain,
          );
          this.rainSnowDay.datasets[0].data.push(chance_of_rain);
          const chance_of_snow = Math.round(currentForecast.hour[i].chance_of_snow,);
          this.rainSnowDay.datasets[1].data.push(chance_of_snow);

          this.cloudDay.labels.push(time);
          const { cloud } = currentForecast.hour[i];
          this.cloudDay.datasets[0].data.push(cloud);

          this.humidityDay.labels.push(time);
          const { humidity } = currentForecast.hour[i];
          this.humidityDay.datasets[0].data.push(humidity);
        }
        const codeSvg = array
                .sort((a, b) => array.filter((v) => v === a).length - array.filter((v) => v === b).length)
                .pop();
        const [, putSunPhase] = this.putInSvg(
                this.Weather.current.is_day,
                codeSvg,
        );
        this.Weather.current.sunMoon.sun_phase = putSunPhase;
      },
      /** Выбор варианта Svg (из какого массива брать) */
      putInSvg(is_day, codePicture) {
        for (const i in this.IncomingPictures) {
          for (const code of this.IncomingPictures[i].codes) {
            if (code === codePicture) {
              if (is_day === 0) {
                return [
                  this.SvgsDay[i].svg,
                  this.SvgsDay[i].name,
                  this.SvgsDay[i].background,
                ];
              }
              return [
                this.SvgsNight[i].svg,
                this.SvgsNight[i].name,
                this.SvgsNight[i].background,
              ];
            }
          }
        }
        return [];
      },
      /** Получение данных о погоде по ближайшим дням */
      getWeekData(data) {
        const { forecast } = data;
        this.Weather.days = [];
        const forecastdays = forecast.forecastday;
        for (let i = 1; i < forecastdays.length; i++) {
          const { code } = forecastdays[i].day.condition;
          const [putSvg, putName] = this.putInSvg(0, code);
          const day = {
            maxTemp: forecastdays[i].day.maxtemp_c,
            minTemp: forecastdays[i].day.mintemp_c,
            svg: putSvg,
            name: putName,
            number: new Date(this.getDayMs(i)).toLocaleDateString(),
          };
          this.Weather.days.push(day);
        }
      },
    },
  };
</script>

<style lang="scss">
  /*вся страница*/
  #app {
    background: linear-gradient(179.1deg, #85bcf1 -1.9%, #d5f3ff 44.9%, #eaf7fc 96.1%);
    min-height: 100vh;
    padding: 0;
    font-family: 'Roboto Condensed', sans-serif;
  }
  /*контейнер*/
  .container {
    padding: 0 !important;
  }
  /*заголовки*/
  h2 {
    text-align: left;
    color: black;
    font-weight: 500 !important;
  }
  h4 {
    font-size: 1.2rem !important;
    text-align: center;
  }
  /*контейнер при ошибки*/
  .container_error {
    padding: 18% 0 0 !important;
  }
  /*блок под текст ошибки*/
  .error {
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    /*заголовки*/
    h1 {
      color: #db4b32;
      margin-left: 55px;
    }
    h1,
    h2 {
      text-align: left;
    }
  }
  /*svg текущей погоды (сегодня)*/
  .mainSvg {
    width: 120%;
    margin-top: -10%;
    margin-left: -10%;
  }
  /*название дня или ночи*/
  .day_name {
    text-align: left;
    margin-bottom: 0 !important;
  }
  /*svg дня или ночи*/
  .sunMoonSvg {
    width: 255%;
    margin: 30% auto 0 -45%;
    display: block;
  }
  /*svg следующих дней (день недели)*/
  .svgDayOfTheWeek {
    width: 185%;
    margin: 30% auto 0 -15%;
    display: block;
  }
  /*svg утро/день/вечер*/
  .twentyFourHoursSvg {
    width: 125%;
    margin: -20% auto 0 -10%;
  }
  /*температура утро/день/вечер*/
  .twentyFourHoursTemp {
    font-size: 1.6rem;
    text-align: center;
    margin-top: -10%;
  }
  /*блок графиков*/
  canvas {
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    margin: 0 0 2% 0;
  }
  /*блок под кратк. инф о сегодняшней погоде*/
  .today {
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    text-align: left;
    height: 125px;
    color: black;
    padding: 15px;
    margin-bottom: 7px;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  /*блок под пол. инф о сегодняшней погоде, блок под погоду утром/днем/вечером*/
  .feels,
  .hours {
    text-align: left;
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    margin-bottom: 3%;
    padding: 15px;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
  }
  /*кратк. инф о сегодняшней погоде*/
  .today_inf p,
  .today_inf {
    color: black;
    text-align: left;
    margin-bottom: 0;
  }
  /*разделительная горизонтальная линия*/
  hr {
    margin-top: 0 !important;
    margin-bottom: 0 !important;
    background: transparent;
    box-shadow: 0 0.5px 0.5px rgba(0, 0, 0, 0.25);
  }

  .now {
    font-size: 2.5rem;
    color: black;
    text-align: center;
  }
  .temp {
    font-size: 2rem;
    line-height: 2rem;
    text-align: center;
    margin-top: -8%;
    margin-bottom: 0;
  }
  /*элемент выбора города*/
  select,
  select:focus {
    background: transparent !important;
    border: none !important;
    outline: none;
  }
  /*пункты элемента выбора города*/
  option {
    background-color: rgba(255, 255, 255, 0.66) !important;
  }
  /*блок под выбор города*/
  .city {
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
  /*блок под значения*/
  .conditions {
    font-size: 0.85rem !important;
    line-height: 1rem;
    text-align: center;
    margin-bottom: 0 !important;
  }
  /*блок под важные значения*/
  .sunMoon {
    margin-top: 25%;
  }
  /*температура у ближайших дней*/
  .darkLight_temp {
    margin-top: 15%;
    text-align: center;
    font-size: 1.5rem;
    margin-bottom: 0 !important;
  }
  /*блоки под ближайшие дни*/
  .day {
    background: rgba(255, 255, 255, 0.66);
    border-radius: 10px;
    color: black;
    box-shadow: 0 2px 3px rgba(0, 0, 0, 0.25);
    padding: 12px 12px 6px;
    /*заголовок*/
    h2 {
      font-size: 1rem !important;
      font-weight: 500;
      margin-bottom: 0 !important;
    }
    /*кратк. инф о сегодняшней погоде*/
    .today_inf p,
    .today_inf {
      font-size: 0.9rem;
    }
  }
  /*параметры сетки бутстрап*/
  .col-3 {
    padding: 0 !important;
  }
  /*параметры сетки бутстрап*/
  .col,
  .col-3 {
    /*блоки под ближайшие дни*/
    .flex-container > .day {
      width: 47%;
      margin: 0 1% 7px;
    }
  }
  /*параметры сетки бутстрап*/
  .col {
    padding: 0 3%;
    /*блоки под диаграммы (графики)*/
    .flex-container > .chart {
      width: 47.8%;
      margin: 0 8px 7px;
    }
  }
  /*блоки под ближайшие дни*/
  .flex-container > .day {
    width: 15.16%;
    height: 125px;
    margin: 0 0.75% 1%;
  }
  /*контейнер, который включает в себя повтор блоков, блок под погоду утром/днем/вечером*/
  .flex-container,
  .hours {
    display: flex;
    flex-wrap: wrap;
  }
  .flex-container > p {
    width: 100%;
    margin: 0;
  }
  /*блоки под погоду утром/днем/вечером*/
  .hours > div {
    width: 31%;
    margin: 0 1%;
  }
  .addRow .col:first-child {
    padding-right: 0 !important;
  }
  .addRow .col:last-child {
    padding-left: 0 !important;
  }

  @media screen and (min-width: 992px) and (max-width: 1200px) {
    /*блок под выбор города*/
    .city {
      width: 40%;
      font-size: 1.2rem;
    }
    /*блоки под диаграммы (графики)*/
    .col .flex-container > .chart {
      width: 47.5%;
    }
    /*температура у ближайших дней*/
    .darkLight_temp {
      font-size: 1.2rem;
      margin-top: 25%;
    }
    /*заголовок у блока под ближайшие дни*/
    .day h2 {
      font-size: 0.9rem !important;
    }
    /*svg следующих дней (день недели)*/
    .svgDayOfTheWeek {
      margin: 50% auto 0 -15%;
    }
    /*svg дня или ночи*/
    .sunMoonSvg {
      margin: 70% auto 0 -45%;
    }
  }
  @media screen and (min-width: 768px) and (max-width: 992px) {
    /*блок под выбор города*/
    .city {
      width: 55%;
      font-size: 1.2rem;
    }
    /*бутстрап сетка*/
    .mainRow {
      flex-direction: column;
      /*параметры сетки бутстрап*/
      .col-3 {
        display: flex;
        max-width: 100%;
        padding: 0 15px !important;
        /*блоки*/
        & > div {
          width: 32%;
          margin: 0 0.75% 1%;
        }
      }
    }
    /*блоки под диаграммы (графики)*/
    .col .flex-container > .chart {
      width: 47.4%;
    }
    /*температура у ближайших дней*/
    .darkLight_temp {
      font-size: 1.2rem;
      margin-top: 25%;
    }
    /*заголовок у блока под ближайшие дни*/
    .day h2 {
      font-size: 0.9rem !important;
    }
    /*svg следующих дней (день недели)*/
    .svgDayOfTheWeek {
      margin: 50% auto 0 -15%;
    }
    /*svg дня или ночи*/
    .sunMoonSvg {
      margin: 70% auto 0 -45%;
    }
  }
  @media screen and (min-width: 500px) and (max-width: 768px) {
    /*блок под выбор города*/
    .city {
      width: 93%;
      font-size: 1.2rem;
    }
    /*бутстрап сетка*/
    .row {
      margin: 0 !important;
    }
    /*бутстрап сетка*/
    .mainRow,
    .addRow {
      flex-direction: column;
      /*параметры сетки бутстрап*/
      .col {
        padding: 0 5px !important;
      }
      /*параметры сетки бутстрап*/
      .col-3 {
        max-width: 100%;
        padding: 0 15px !important;
      }
    }
    /*svg текущей погоды (сегодня)*/
    .mainSvg {
      width: 60%;
      float: left;
      display: block;
      margin: 10% auto 0;
    }
    .temp {
      margin-top: 15%;
    }
    /*svg утро/день/вечер*/
    .twentyFourHoursSvg {
      width: 65%;
      margin: -10% auto 5% 20%;
    }
    /*блоки под ближайшие дни*/
    .col .flex-container > .day,
    .col-3 .flex-container > .day {
      width: 48%;
      margin: 0 1% 7px;
    }
    /*блоки под диаграммы (графики)*/
    .col .flex-container > .chart {
      width: 100%;
    }
    /*температура у ближайших дней*/
    .darkLight_temp {
      font-size: 1.2rem;
      margin-top: 15%;
    }
    /*заголовок у блоков под ближайшие дни*/
    .day h2 {
      font-size: 0.9rem !important;
    }
    /*svg следующих дней (день недели)*/
    .svgDayOfTheWeek {
      width: 145%;
      margin: 15% auto 0 auto;
    }
    /*svg дня или ночи*/
    .sunMoonSvg {
      margin: 25% auto 0 auto;
      width: 155%;
    }
    /*блок под важные значения*/
    .sunMoon {
      margin-top: 20%;
    }
  }
  @media screen and (max-width: 500px) {
    .city {
      width: 93%;
      font-size: 1.2rem;
    }
    /*бутстрап сетка*/
    .row {
      margin: 0 !important;
    }
    /*бутстрап сетка*/
    .addRow {
      padding: 0 5px;
    }
    /*бутстрап сетка*/
    .mainRow,
    .addRow,
    .darkLight {
      flex-direction: column;
      /*параметры сетки бутстрап*/
      .col {
        padding: 0 5px !important;
      }
      /*параметры сетки бутстрап*/
      .col-3,
      .col-4,
      .col-5 {
        max-width: 100%;
        padding: 0 15px !important;
      }
    }
    /*svg текущей погоды (сегодня)*/
    .mainSvg {
      width: 80%;
      display: block;
      margin: -10% auto 0;
    }
    .temp {
      margin-top: 0;
    }
    /*svg утро/день/вечер*/
    .twentyFourHoursSvg {
      width: 65%;
      margin: -10% auto 5% 20%;
    }
    /*блоки под ближайшие дни*/
    .col .flex-container > .day,
    .col-3 .flex-container > .day {
      width: 48%;
      margin: 0 1% 7px;
    }
    /*блоки под диаграммы (графики)*/
    .col .flex-container > .chart {
      width: 99%;
      padding: 0 10px 7px;
      margin: 0;
    }
    /*температура у ближайших дней*/
    .darkLight_temp {
      font-size: 1.2rem;
      margin-top: -4%;
    }
    /*заголовок у блоков под ближайшие дни*/
    .day h2 {
      font-size: 0.85rem !important;
    }
    /*svg дня или ночи, svg следующих дней (день недели)*/
    .sunMoonSvg,
    .svgDayOfTheWeek {
      margin: 0 auto 0 auto;
      width: 65%;
    }
    /*блок под важные значения*/
    .sunMoon {
      margin-top: 0;
    }
    /*блок под значения*/
    .conditions {
      margin-top: -2%;
    }
  }
</style>
