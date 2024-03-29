<template>
  <div>
    <Loading v-if="loading" />
    <div v-else id="main" :class="isDay ? 'day' : 'night'">
      <div class="container my-5" style="max-width: 400px; min-width: 360px">
        <h1 class="title text-center text-white text-uppercase">
          <span class="smash">Weather</span> in
        </h1>
        <form
          v-on:submit.prevent="getWeather"
          class="search-location position-relative"
        >
          <input
            type="text"
            class="form-control text-muted form-rounded p-3 shadow-sm"
            placeholder="What City?"
            v-model="citySearch"
            autocomplete="off"
          />
          <span role="button" v-on:click="getWeather" class="btn-weather">
            <i class="fa-solid fa-bolt"></i>
          </span>
        </form>
        <Transition name="bounce">
          <p class="text-center my-3 city-not_found" v-if="cityFound">
            No city found
          </p>
        </Transition>

        <Transition name="fadeIn">
          <div
            class="card rounded my-3 shadow-lg back-card overflow-hidden"
            v-if="visible"
          >
            <!-- weather animation container -->
            <div>
              <div icon="sunny" v-if="clearSky" data-label="Sunny">
                <span class="sun"></span>
              </div>

              <div icon="snowy" v-if="snowy" data-label="Chilly">
                <ul>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                </ul>
              </div>

              <div icon="stormy" v-if="stormy" data-label="Soggy">
                <span class="cloud"></span>
                <ul>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                  <li></li>
                </ul>
              </div>
              <div icon="cloudy" v-if="cloudy" data-label="Perfect">
                <span class="cloud"></span>
                <span class="cloud"></span>
              </div>
              <div icon="nightmoon" v-if="clearNight" data-label="Cool!">
                <span class="moon"></span>
                <span class="meteor"></span>
              </div>
            </div>

            <!-- Top of card starts here -->
            <div class="card-top text-center" style="margin-bottom: 15rem">
              <div class="city-name my-3">
                <p>{{ weather.cityName }}</p>
                <span>...</span>
                <p class="">{{ weather.country }}</p>
              </div>
            </div>
            <!-- top of card ends here -->

            <!--card middle body, card body used cos I want to just update the innerHTML -->
            <div class="card-body">
              <!-- card middle starts here -->
              <div class="card-mid">
                <div class="row">
                  <div class="col-12 text-center temp">
                    <span>{{ weather.temperature }}&deg;C</span>
                    <p class="my-4">{{ weather.description }}</p>
                  </div>
                </div>
                <div class="row">
                  <div class="col d-flex justify-content-between px-5 mx-5">
                    <p>
                      <img src="./assets/down.svg" alt="" />
                      {{ weather.lowTemp }}&deg;C
                    </p>
                    <p>
                      <img src="./assets/up.svg" alt="" />
                      {{ weather.highTemp }}&deg;C
                    </p>
                  </div>
                </div>
              </div>
              <!-- card middle ends here -->

              <!-- card bottom starts here -->
              <div class="card-bottom px-5 py-4 row">
                <div class="col text-center">
                  <p>{{ weather.feelsLike }}&deg;C</p>
                  <span>Feels like</span>
                </div>
                <div class="col text-center">
                  <p>{{ weather.humidity }}%</p>
                  <span>humidity</span>
                </div>
              </div>

              <!-- card bottom ends here -->
            </div>
          </div>
        </Transition>
      </div>
    </div>
  </div>
</template>

<script>
import Loading from "../src/components/Loading.vue";
export default {
  name: "App-Vue",
  data() {
    return {
      loading: true,
      cityFound: false,
      visible: false,
      stormy: false,
      cloudy: false,
      clearSky: false,
      clearNight: false,
      snowy: false,
      isDay: true,
      citySearch: "",
      weather: {
        cityName: "Gwarinpa",
        country: "NG",
        temperature: 12,
        description: "Clouds everywhere",
        lowTemp: "19",
        highTemp: "30",
        feelsLike: "20",
        humidity: "55",
      },
    };
  },
  methods: {
    getWeather: async function () {
      console.log(this.citySearch);
      const key = "ffe228798fae520e9d74dc8370044c6a";
      const baseURL = `https://api.openweathermap.org/data/2.5/weather?q=${this.citySearch}&appid=${key}&units=metric`;

      //fetch weather
      try {
        const response = await fetch(baseURL);
        const data = await response.json();
        console.log(data);
        this.citySearch = "";
        this.weather.cityName = data.name;
        this.weather.country = data.sys.country;
        this.weather.temperature = Math.round(data.main.temp);
        this.weather.description = data.weather[0].description;
        this.weather.lowTemp = Math.round(data.main.temp_min);
        this.weather.highTemp = Math.round(data.main.temp_max);
        this.weather.feelsLike = Math.round(data.main.feels_like);
        this.weather.humidity = Math.round(data.main.humidity);

        const timeOfDay = data.weather[0].icon;

        ///check for time of day
        if (timeOfDay.includes("n")) {
          this.isDay = false;
        } else {
          this.isDay = true;
        }

        const mainWeather = data.weather[0].main;
        //check weather animations
        if (mainWeather.includes("Clouds")) {
          this.stormy = false;
          this.cloudy = true;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clouds")) {
          this.stormy = false;
          this.cloudy = true;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (
          mainWeather.includes("Thunderstorm") ||
          mainWeather.includes("Rain")
        ) {
          this.stormy = true;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clear") && this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = true;
          this.clearNight = false;
          this.snowy = false;
        }
        if (mainWeather.includes("Clouds") && !this.isDay) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = true;
          this.snowy = false;
        }
        if (mainWeather.includes("Snow")) {
          this.stormy = false;
          this.cloudy = false;
          this.clearSky = false;
          this.clearNight = false;
          this.snowy = true;
        }

        this.visible = true;
        this.cityFound = false;
      } catch (error) {
        console.log(error);
        this.cityFound = true;
        this.visible = false;
      }
    },
  },
  created() {
    setTimeout(() => {
      this.loading = false;
    }, 500);
  },
  components: {
    Loading,
  },
};
</script>
<style>
.bounce-enter-active {
  animation: bounce-in 0.5s;
}
.fadeIn-enter-active {
  animation-delay: 0.5s;
  animation: fadeIn 0.8s;
}
.fadeIn-leave-active {
  animation: fadeIn 0.5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.25);
  }
  100% {
    transform: scale(1);
  }
}
@keyframes fadeOut {
  100% {
    transform: translate(120%, 0px);
    opacity: 0;
  }
}
@keyframes fadeIn {
  0% {
    transform: translateY(100%);
  }
  100% {
    transform: translateY(0%);
  }
}
</style>
