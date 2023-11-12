<template>
  <div id="app">
    <b-container>
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
            <component  :is="svg" width="90%"/>
            <div class="now">
                <div class="temp">+3°</div>
              <div>Moscow</div>
            </div>
            <hr>
            <div class="number">
               Feels like: +3°
               <br>
               Humidity: 51%
               <br>
               Wind: 9 km/h
               </div>
          </div>
        </b-col>
        <b-col>
          <div class="flex-container" >
            <div class="day" v-for="(day, index) in Weather" :key="index">
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
                      {{day.day}}°
                    </div>
                    <div class="light">
                      {{day.night}}°
                    </div>
                  </div>
                </b-col>
              </b-row>

            </div>
          </div>
        </b-col>
      </b-row>
<!--      <div class="flex-container">-->
<!--        <div class="svg" v-for="(day, index) in Svgs" :key="index">-->
<!--          <component :key="index" :is="day.svg" width="145%" class="svgDayOfTheWeek"/>-->
<!--        </div>-->
<!--      </div>-->
    </b-container>
  </div>
</template>

<script>
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
export default {
  name: 'App',
  components: {},
  data(){
    return{
      svg:partlyRaining,
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
          name:'partly raining and snowing',
          svg: partlyRainingAndSnowing,
          desk:'Переменный дождь и снег'
        },
        {
          name:'partly raining',
          svg: partlyRaining,
          desk:'Переменный дождь'
        },
        {
          name:'partly snowing',
          svg: partlySnowing,
          desk:'Переменный снег'
        },
        {
          name:'drizzling rain',
          svg:drizzlingRain,
          desk:'Моросящий дождь'
        },
        {
          name:'rain',
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
      ],
      Weather:[
        {
          name:'rain',
          svg:rain,
          day:'+7',
          night:'+4'
        },
        {
          name:'drizzling',
          svg:drizzlingRain,
          day:'+4',
          night:'+1'
        },
        {
          name:'cloudy',
          svg:cloudy,
          day:'+2',
          night:'0'
        },
        {
          name:'rain',
          svg:rain,
          day:'+2',
          night:'0'
        },
        {
          name:'rain',
          svg:rain,
          day:'+2',
          night:'0'
        },
        {
          name:'rain',
          svg:rain,
          day:'+2',
          night:'0'
        },
      ]
    }
  },
  methods:{
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
    }
  }
}
</script>

<style lang="scss">
  .today{
    background: rgba(255, 255, 255, 0.85);
    border-radius: 20px;
    padding: 15px;
    box-shadow: 0px 2px 3px rgba(0, 0, 0, 0.25);
  }
  .today_time{
    color: black;
    text-align: left;
    font-size: 1.3rem;
  }
#app {
  //background: linear-gradient(178.6deg, rgb(20, 36, 50) 11.8%, rgb(91, 107, 121) 83.8%);
  background: linear-gradient(179.1deg, rgb(203 180 180) -1.9%, rgb(187 153 152) 44.9%, rgb(127 147 184) 96.1%);
  min-height: 100vh;
  text-align: center;
  color: white;
  padding: 50px 0;
  font-family: 'Roboto Condensed', sans-serif;
}
hr{
  //background: black;
  margin-top: .5rem !important;
  margin-bottom: .5rem !important;
  background: transparent;
  box-shadow: 0px .5px .5px rgba(0, 0, 0, 0.25);
}
  h2{
    text-align: left;
    color: black;
    font-weight: 500 !important;
  }

.time{
  font-size: 3.2rem;
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
.dayOfTheWeek{
  font-size: 2rem;
  text-align: center;
}
.number{
  font-size: 1.3rem;
  text-align: left;
  color: black;
}
.light, .dark{
  width: 45px;
  height: 45px;
  background: #f1fbff;
  padding: 5px;
  font-size: 1.8rem;
  text-align: center;
  border-radius: 10px;
  box-shadow: 0px 2px 3px rgba(0, 0, 0, 0.25);
}
.dark{
  margin-left: 20px;
}
.light{
  margin-top: -5px;
}
.degrees{
  margin-top: 10px;
  font-size: 2rem;
  text-align: left;
}
  .current{
    background: rgba(248, 248, 248, 0.25);
    backdrop-filter: blur(10px);
    border-radius: 15px;
    border: 1px solid rgba(43, 43, 43, 0.568);
    padding: 15px;
    margin-bottom: 40px;
  }
  .day{
    background: rgba(255, 255, 255, 0.85);
    border-radius: 20px;
    color: black;
    box-shadow: 0px 2px 3px rgba(0, 0, 0, 0.25);
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
  .day:last-child{
    border-right: none;
  }
  .flex-container > .day{
    width: 43%;
    margin: 0 1% 2%;
  }
  .flex-container > .svg{
    width: 15%;
    margin: 0 1% 2%;
  }
  .flex-container{
    display: flex;
    flex-wrap: wrap;
  }
</style>
