<template>
  <div id="app">
    <b-container>
      <div class="current">
        <h1>Moscow</h1>
        <hr>
        <b-row>
          <b-col>
            <component :key="index" :is="svg" width="100%"/>
          </b-col>
          <b-col>
            <div class="time">21:43</div>
            <hr>
            <b-row>
              <b-col cols="4">
                <div class="now">+3°</div>
              </b-col>
              <b-col >
                <div class="number">
                  Feels like: +3°
                  <br>
                  Humidity: 51%
                  <br>
                  Wind: 9 km/h
                </div>
              </b-col>
            </b-row>
          </b-col>
        </b-row>
        <hr>
        <div class="flex-container" >
          <div class="day" v-for="(day, index) in Weather" :key="index">
            <div class="dayOfTheWeek">{{day.dayOfTheWeek}}</div>
            <div class="number">{{day.number}} oct</div>
            <component :key="index" :is="day.svg" width="100%" class="svgDayOfTheWeek"/>
            <div class="degrees">
              d: {{day.day}}°
              <br>
              n: {{day.night}}°
            </div>

          </div>
        </div>
      </div>
    </b-container>
  </div>
</template>

<script>
  import sunny from "./components/icons/sunny";
  import rain from "./components/icons/rain";
  import drizzing from "./components/icons/drizzing";
  import sunnyRain from "@/components/icons/sunnyRain";
  import cloudy from "@/components/icons/cloudy";

export default {
  name: 'App',
  components: {},
  data(){
    return{
      svg:sunnyRain,
      Weather:[
        {
          dayOfTheWeek:'Mon',
          number:'23',
          svg:rain,
          day:'+7',
          night:'+4'
        },
        {
          dayOfTheWeek:'Tue',
          number:'24',
          svg:drizzing,
          day:'+4',
          night:'+1'
        },
        {
          dayOfTheWeek:'Wed',
          number:'25',
          svg:cloudy,
          day:'+2',
          night:'+0'
        },
        {
          dayOfTheWeek:'Thu',
          number:'26',
          svg:rain,
          day:'+2',
          night:'0'
        },
        {
          dayOfTheWeek:'Fri',
          number:'27',
          svg:rain,
          day:'+2',
          night:'0'
        },
        {
          dayOfTheWeek:'Sat',
          number:'28',
          svg:rain,
          day:'+2',
          night:'0'
        },
        {
          dayOfTheWeek:'Sun',
          number:'29',
          svg:rain,
          day:'+5',
          night:'+2'
        }
      ]
    }
  }
}
</script>

<style>
#app {
  background: linear-gradient(178.6deg, rgb(20, 36, 50) 11.8%, rgb(91, 107, 121) 83.8%);
  height: 100vh;
  text-align: center;
  color: white;
  padding: 50px 0;
  font-family: 'Roboto Condensed', sans-serif;

}
hr{
  background: rgba(255, 255, 255, 0.75);
  margin-top: .5rem !important;
  margin-bottom: .5rem !important;
}
.time{
  font-size: 3.2rem;
}
.now{
  font-size: 5.5rem;
  margin-left: 60px;
  text-align: center;
}
.svgDayOfTheWeek{
  margin-top: 10px;
}
.dayOfTheWeek{
  font-size: 2rem;
  text-align: center;
}
.number{
  font-size: 1.6rem;
  text-align: center;
}
.degrees{
  margin-top: 10px;
  font-size: 2rem;
  text-align: left;
}
.current, .day{

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
    border-right: 1px solid rgba(255, 255, 255, 0.55);
  }
  .day:last-child{
    border-right: none;
  }
  .flex-container > .day{
    width: 12.2%;
    margin: 2% 1% 3%;
  }
  .flex-container{
    display: flex;
    flex-wrap: wrap;
  }
</style>
