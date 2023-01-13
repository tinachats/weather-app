<template>
  <main id="content" :class="main" v-if="typeof main != 'undefined'">
    <!-- Today's Weather -->
    <div class="container">
      <!-- Nav -->
      <div class="row align-items-center justify-content-between py-3 gx-3">
        <div class="col-2 d-flex justify-content-start">
          <button class="btn btn-light btn-sm rounded-circle shadow-sm">
            <i class="bi-list app-color"></i>
          </button>
        </div>
        <div class="col-8">
          <div class="form-group">
            <i class="bi-search app-color"></i>
            <input
              type="search"
              name="query"
              id="query"
              class="form-control form-control-sm rounded-pill app-color"
              v-model="query"
              placeholder="Search any city"
              @keypress="searchWeather"
            />
          </div>
        </div>
        <div class="col-2 d-flex justify-content-end">
          <button
            class="btn btn-light btn-sm rounded-circle shadow-sm"
            @click="getWeather"
            data-bs-toggle="tooltip"
            data-bs-trigger="hover"
            data-bs-placement="bottom"
            title="Get your current weather"
          >
            <i class="bi-geo-alt-fill app-color"></i>
          </button>
        </div>
      </div>
      <!-- /.Nav -->

      <!-- Weather Info -->
      <div
        class="d-flex justify-content-center"
        v-if="typeof main != 'undefined'"
      >
        <div class="d-block text-center">
          <h1 class="text-capitalize ls-1 mb-0">{{ city }}, {{ country }}</h1>
          <h6 class="small text-capitalize">{{ todaysDate() }}</h6>
        </div>
      </div>
      <div class="d-md-flex align-items-center justify-content-center">
        <div class="d-block text-center">
          <img
            :src="icon"
            :alt="description"
            height="250"
            width="250"
            class="weather-icon pl-1"
          />
        </div>
        <div class="d-block text-center lh-0">
          <h2 class="display-1 fw-bold">{{ temp }}°C</h2>
          <h6 class="text-capitalize mb-0">{{ description }}</h6>
          <p class="small-xs py-0">Weather now</p>
        </div>
      </div>
      <hr class="my-0" />
      <div class="d-flex align-items-center justify-content-around">
        <div class="d-block text-center">
          <i class="bi-droplet-half"></i>
          <h5 class="fs-6 fw-bold mb-0">{{ humidity }}%</h5>
          <h6 class="small-xs">Humidity</h6>
        </div>
        <div class="d-block text-center">
          <i class="bi-wind"></i>
          <h5 class="fs-6 fw-bold mb-0">{{ wind }}m/s</h5>
          <h6 class="small-xs">Wind</h6>
        </div>
        <div class="d-block text-center">
          <i class="bi-eye"></i>
          <h5 class="fs-6 fw-bold mb-0">{{ visibility }}km</h5>
          <h6 class="small-xs">Visibility</h6>
        </div>
      </div>
      <!-- /.Weather Info -->
    </div>
    <!-- /.Today's Weather -->
  </main>

  <section class="container my-3">
    <h5 class="fs-6 fw-bold text-capitalize app-color">weekly weather</h5>
    <!-- Week's Weather -->
    <div class="nav-scroller mb-3">
      <div class="container">
        <nav class="nav d-flex justify-content-between" id="weather-forecast">
          <!-- Weekly weather will be shown here -->
        </nav>
      </div>
    </div>
    <!-- /.Week's Weather -->
  </section>

  <input type="hidden" name="lat" id="lat" v-model="lat" />
  <input type="hidden" name="lon" id="lon" v-model="lon" />
</template>

<script>
const days = [
  "Sunday",
  "Monday",
  "Tuesday",
  "Wednesday",
  "Thursday",
  "Friday",
  "Saturday",
];
const months = [
  "January",
  "February",
  "March",
  "April",
  "May",
  "June",
  "July",
  "August",
  "September",
  "October",
  "November",
  "December",
];

export default {
  data() {
    return {
      api_key: "87e34eb71fecbe55690036e301e8198c",
      base_url: "https://api.openweathermap.org/data/2.5/",
      city: "---------",
      country: "--",
      date: this.todaysDate(),
      description: "",
      humidity: "--",
      path: '../src/assets/weather-icons/',
      icon: this.path + "09d.svg",
      lat: "",
      lon: "",
      main: "",
      output: "",
      query: "",
      temp: "--",
      visibility: "--",
      wind: "--",
    };
  },
  methods: {
    pad(n, s = String(n)) {
      return s.length < 2 ? `0${s}` : s;
    },
    stringToDay(date) {
      const dt = new Date(date).getDay();
      const day = days[dt];
      return day;
    },
    todaysDate() {
      let dt = new Date();
      let day = days[dt.getDay()];
      let date = this.pad(dt.getDate());
      let month = months[dt.getMonth()];
      let year = dt.getFullYear();

      return `${day} ${date} ${month}, ${year}`;
    },
    getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((pos) => {
          document.getElementById("lat").value = pos.coords.latitude;
          document.getElementById("lon").value = pos.coords.longitude;
          console.log("Coordinates have been set.");
        });
      } else {
        alert("Geocode is not supported by your browser.");
      }
    },
    getWeather() {
      this.lat = document.getElementById("lat").value;
      this.lon = document.getElementById("lon").value;

      fetch(
        `${this.base_url}forecast?lat=${this.lat}&lon=${this.lon}&units=metric&appid=${this.api_key}`
      )
        .then((res) => res.json())
        .then((data) => {
          console.log(data);
          this.city = data.city.name;
          this.country = data.city.country;
          let forecast = data.list;

          // Get current weather
          this.humidity = forecast[0].main.humidity;
          this.icon = this.path + forecast[0].weather[0].icon + '.svg';
          this.main = forecast[0].weather[0].main.toLowerCase();
          this.temp = Math.round(forecast[0].main.temp);
          this.visibility = Math.round(forecast[0].visibility / 1000);
          this.wind = Math.round(forecast[0].wind.speed);

          // Get weather forecast for the next 5 days
          for (var i = 0; i < forecast.length; i++) {
            this.output += `
            <a class="scrolling-weather-card" href="#">
              <div
                class="d-flex flex-column justify-content-between align-items-center"
              >
                <h5 class="fw-bold small-xs">${Math.round(forecast[i].main.temp)}°C</h5>
                <img
                  src="../src/assets/weather-icons/${forecast[i].weather[0].icon}.svg"
                  alt="icon"
                  width="30"
                  height="30"
                />
                <h6 class="small-xs text-capitalize mb-0">${this.stringToDay(forecast[i].dt_txt)}</h6>
              </div>
            </a>
          `;
          }
          document.getElementById('weather-forecast').innerHTML = this.output;
        });
    },
  },
  created() {
    this.getLocation(),
      setTimeout(() => {
        this.getWeather();
      }, 1000);
  },
};
</script>

<style>
main.clear,
main#content {
  background-image: url("../src/assets/background-images/clear-bg.gif");
}
main.snow {
  background-image: url("../src/assets/background-images/snow2-bg.gif") !important;
}
main.drizzle,
main.rain {
  background-image: url("../src/assets/background-images/animated-rain-bg.gif") !important;
}
main.windy {
  background-image: url("../src/assets/background-images/animated-windy-bg.gif") !important;
}
main.thunderstorm,
main.clouds {
  background-image: url("../src/assets/background-images/clouds-storm.gif") !important;
}
main.sunny {
  background-image: url("../src/assets/background-images/sunny-day-bg.gif") !important;
}
main.mist,
main.dust,
main.haze,
main.smoke,
main.foggy {
  background-image: url("../src/assets/background-images/foggy-bg.gif") !important;
}
</style>
