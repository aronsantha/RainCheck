<template>
  <div id="app">
    <div class="screen" :style="bgGradient">
      <div class="top-holder" style="order: 1">
        <img :src="bar" id="status-bar" alt="Status bar." />
        <div id="time">{{ statusClock }}</div>
        <div id="search">
          <div>
            <input
              type="text"
              v-model="cityName"
              @keyup.enter="getWeather()" 
              placeholder="Search for city"
              id="search-box"
            />
            <button id="submit" @click="getWeather()">Go!</button>
          </div>
        </div>
      </div>
      <div style="order: 2" v-show="visible">
        <p id="location">
          <b>{{ location }}</b> ({{ country }})
        </p>
       </div>
      <div class="display-current" style="order: 3" v-show="visible">
        <p id="today">Today</p>
        <p id="temp">{{ temp }}°C</p>
        <p id="sky">{{ sky }}</p>
        <p id="feels-like">Feels like: {{ feelsLike }}°C</p>
       </div>
 

      <div class="display-more" style="order: 4" v-show="visible">
        <section style="display: flex">
          <div class="half-left">
            <ul id="list-style">
              <li>Sunrise: {{ sunRise }}</li>
              <li>Wind: {{ windKmh }} km/h</li>
              <li>Lowest: {{ minTemp }}°C</li>
            </ul>
          </div>
          <div class="half-right">
            <ul id="list-style">
              <li>Sunset: {{ sunSet }}</li>
              <li>Humidity: {{ humidity }}%</li>
              <li>Highest: {{ maxTemp }}°C</li>
            </ul>
          </div>
        </section>
      </div>
      <div id="home-button"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  data() {
    return {
      bar: require("./assets/statusbar.png"),
      bgGradient: "",
      bgObject: {
        morning: {
          background:
            "linear-gradient(to bottom, #0a1831 20%,#6567A4 75%,#cd82a0 100%)",
        },
        day: {
          background:
            "linear-gradient(to bottom, #0e66c4 25%, #2d97e3 75%, #b0d6f5 100%)",
        },
        evening: {
          background:
             "linear-gradient(to bottom, #3b1727 30%,#f85f18 90%,#ffae00 100%)",
        },
        night: {
          background:
            "linear-gradient(to bottom, #020b1c 0%,#0a1831 50%,#083464 100%)",
        },
      },
      error: false,
      cityName: "",
      temp: "",
      location: "",
      country: "",
      localTime: "",
      sky: "",
      sunRise: "",
      sunSet: "",
      windMs: "", // wind meter / sec
      windKmh: "", // wind km/ h
      humidity: "",
      feelsLike: "",
      minTemp: "",
      maxTemp: "",
      statusClock: "",
      hours: "",
      visible: false,
    };
  },
  created() {
    setInterval(this.getUserTime, 1000);
    setInterval(this.setGradient, 1000);
  },
  mounted() {
    this.getLocation();
  },
  methods: {
    getWeather(param = this.cityName) {
      fetch(
        "https://api.openweathermap.org/data/2.5/weather?q=" +
          param +
          "&units=metric" +
          "&appid=" +
          "03a96a80c8826098844de3a67f12ebdb"
      )
        .then((response) => response.json())
        .then((data) => {
          this.temp = Math.round(data["main"]["temp"]);
          this.location = data["name"];
          this.country = data["sys"]["country"];
          // this.localTime = 
          var unixRise = new Date(data["sys"]["sunrise"]*1000) // get timestamp, convert to milliseconds
          this.sunRise = (unixRise.getHours() < 10 ? "0" : "") // extract hours (add 0 if necessary)
                          + unixRise.getHours()
                          + ":"
                          + (unixRise.getMinutes() < 10 ? "0" : "") // extract minutes (add 0 if necessary)
                          + unixRise.getMinutes();
          var unixSet = new Date(data["sys"]["sunset"]*1000) // get timestamp, convert to milliseconds
          this.sunSet = (unixSet.getHours() < 10 ? "0" : "") // extract hours (add 0 if necessary)
                          + unixSet.getHours()
                          + ":"
                          + (unixSet.getMinutes() < 10 ? "0" : "") // extract minutes (add 0 if necessary)
                          + unixSet.getMinutes();
          
          this.sky = data["weather"][0]["description"];
          this.windMs = data.wind.speed;
          this.windKmh = Math.round(this.windMs * 3.6);
          this.humidity = Math.round(data.main.humidity);
          this.feelsLike = Math.round(data["main"]["feels_like"]);
          this.minTemp = Math.round(data["main"]["temp_min"]);
          this.maxTemp = Math.round(data["main"]["temp_max"]);
          this.visible = true;
          this.cityName = ""; // CLEAR FORM AFTER SUBMIT
        })
        .catch((error) => {
          console.log(error);
          this.error = "Enter valid city!";
          alert(this.error);
        });
    },

    getLocation() {
      fetch(
        "https://api.ipdata.co?api-key=cbed6cfa309f2778a694bb4b27753c62caf4f744b481adc258bb4666"
      )
        .then((response) => response.json())
        .then((data) => {
          console.log(data.city);
          this.getWeather(data.city);
        });
    },

    getUserTime() {
      var date = new Date();
      this.hours = date.getHours();
      this.statusClock =
        date.getHours() +
        ":" +
        (date.getMinutes() < 10 ? "0" : "") +
        date.getMinutes();
    },

    setGradient() {
      if (this.hours > 4 && this.hours < 8) {
        this.bgGradient = this.bgObject.morning;
      } else if (this.hours >= 8 && this.hours < 18) {
        this.bgGradient = this.bgObject.day;
      } else if (this.hours >= 18 && this.hours <= 19) {
        this.bgGradient = this.bgObject.evening;
      } else {
        this.bgGradient = this.bgObject.night;
      }
    },
  },
};
</script>

<style>
#app {
  text-align: center;
  color: #ffffff;
  margin-top: 60px;
  text-shadow: 0px 0px 4px #00000044;
}

#app,
body,
#search-box,
#submit {
  font-family: "Rubik", Helvetica, Arial, sans-serif;
}

body {
  background-color: #ffffff;
}

.screen {
  position: absolute;
  width: 375px;
  height: 760px;
  top: 50px;
  left: 50px;
  top: 3vw;
  left: 50%;
  transform: translateX(-50%);
  overflow: hidden;
  border-radius: 40px;
  display: flex;
  flex-direction: column;
  box-shadow: 0px 0px 0px 11px #000000, 0px 0px 0px 17px #292929,
    0px 0px 0px 21px #111;
}

.top-holder {
  background-color:rgba(80, 80, 80, 0.609);
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

#status-bar {
  width: 100%;
  order: 1;
  background-color: rgb(0, 0, 0);
}

#time {
  position: absolute;
  justify-content: left;
  left: 8%;
  top: 1.7%;
  font-size: 14px;
  color: white;
  font-weight: 400;
  letter-spacing: 0.05em;
  text-shadow: none;
}

#search {
  background-color: transparent;
  order: 2;
  width: 100%;
  padding-top: 5%;
  padding-bottom: 5%;
  box-sizing: border-box;
}

#search-box {
  height: 45px;
  width: 75%;
  border: none;
  border-radius: 10px 0px 0px 10px;
  box-sizing: border-box;
  position: relative;
  text-align: center;
  letter-spacing: 0.03em;
  font-size: 18px;
  background-color: #ffffffe8;
  color: #6a6a6a;
}

#search-box:focus {
  outline: none;
  border: 3px solid #000000b0;
;
  color: #000000b0;
}

#submit {
  width: 15%;
  height: 45px;
  position: relative;
  border-radius: 0px 10px 10px 0px;
  border: none;
  text-align: center;
  letter-spacing: 0.03em;
  font-size: 18px;
  background-color: #000000b0;
  color: white;
}

#location {
  text-transform: uppercase;
  font-size: 33px;
  padding: 0px 5%;
  margin: 20px 0px 0px 0px;
  overflow-wrap: break-word;  /* break word if too long */
}

.display-current {
  background-color: transparent;
  display: flex;
  flex-direction: column;
  width: 100%;
  padding: 0px 20px;
  box-sizing: border-box;
  margin: auto;
}



#today {
  font-size: 45px;
  font-weight: 300;
  margin: 0px 0px 0px 0px;
  letter-spacing: 0.01em;
}

#temp {
  margin: 0px;
  font-size: 110px;
  font-weight: 300;
  margin-bottom: 0px;
  letter-spacing: 0.001em;
}

#sky {
  margin: 0px 0px 15px 0px;
  text-transform: capitalize;
  font-size: 20px;
  line-height: 20px;
}

#feels-like {
  margin: 0px;
  font-size: 20px;
  line-height: 25px;
}

.display-more {
  background-color: transparent;
  display: flex;
  flex-direction: column;
  height: 20%;
  width: 100%;
  font-size: 20px;
  box-sizing: border-box;
  width: 100%;
  padding: 0px 30px;
  margin-top: 5px;
  margin-bottom: 30px;
}

.half-left {
  background-color: transparent;
  width: 50%;
  text-align: left;
  padding-right: 20px;
}

.half-right {
  background-color: transparent;
  width: 50%;
  text-align: left;
  padding-left: 20px;
}

#list-style {
  list-style: none;
  padding-left: 0;
  line-height: 180%;
}

#home-button {
  position: absolute;
  bottom: 20px;
  left: 33%;
  height: 5px;
  width: 33%;
  background-color: #ffffff4b;
  border-radius: 20px;
}

@media only screen and (min-width: 375px) and (max-width: 800px) {
  #status-bar {
    display: none;
  }  
  #home-button {
    display: none;
  }
  #time {
    display: none;
  }
  .screen {
    box-shadow: none;
    border-radius: 0px;
    top: 0px;
    left: 50%;
    right: 0px;
    transform: translateX(-50%);
    height: 100%;
    min-height: 650px;
  }
  
  .top-holder {
    width: 100%;
  }

  #search {
    padding-top: 15px;
    padding-bottom: 15px;
  }
}

@media only screen and (min-width: 0px) and (max-width: 374px) {
  #status-bar {
    display: none;
  }
  #home-button {
    display: none;
  }
  #time {
    display: none;
  }
 
 .screen {
    border-radius: 0px;
    top: 0px;
    width: 100%;
    height: 100%;
    min-height: 550px;
   }

  #search {
    padding-top: 10px;
    padding-bottom: 10px;
  }

  #search-box {
    height: 40px;
    font-size: 16px;
  }

  #submit {
    height: 40px;
    font-size: 16px;
}

  #location {
    font-size: 28px;
    margin: 15px 0px 0px 0px;  
  }

  #today {
    font-size: 40px;
  }

  #temp {
    font-size: 100px;
  }

#sky {
  font-size: 19px;
  line-height: 19px;
}

#feels-like {
  font-size: 19px;
  line-height: 19px;
}

.display-more {
  width: 70%;
  font-size: 18px;
  width: 100%;
  padding: 0px 18px;
  margin-top: 5px;
  margin-bottom: 15px;
}

}
</style>