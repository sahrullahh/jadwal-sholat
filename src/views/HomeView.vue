<template>
  <div
    class="home p-5 text-left text-white bg-image bg-fixed bg-no-repeat bg-center font-roboto h-screen backdrop-blur-sm"
  >
    <div class="container max-w-5xl mx-auto lg:mt-20 mt-10">
      <h1 class="text-5xl mt-2 font-medium">{{ this.timeNow }}</h1>
      <h2 class="text-lg mt-2">{{ this.dayName }}</h2>
      <h2 class="text-sm mt-2 cursor-pointer" @click="changePosition">
        <i class="bi bi-geo-alt-fill mr-2"> </i>
        {{ this.locality }}, {{ this.principalSubdivision }}
        <i class="bi bi-arrow-clockwise refresh"></i>
      </h2>

      <div class="wrapper mt-20">
        <h2>Jadwal sholat</h2>
        <div class="grid lg:grid-cols-5 grid-cols-2 gap-5 mt-5">
          <div class="text-2xl" v-for="item in jadwal" :key="item.name">
            <div v-if="item.name == 'Shubuh'">
              <i class="bi bi-brightness-alt-high mr-2 text-lg"></i>
            </div>
            <div v-else-if="item.name == 'Dzuhur'">
              <i class="bi bi-sun mr-2 text-lg"></i>
            </div>
            <div v-else-if="item.name == 'Ashr'">
              <i class="bi bi-cloud-sun mr-2 text-lg"></i>
            </div>
            <div v-else-if="item.name == 'Maghrib'">
              <i class="bi bi-moon mr-2 text-lg"></i>
            </div>

            <div v-else-if="item.name == 'Isya'">
              <i class="bi bi-cloud-moon mr-2 text-lg"></i>
            </div>
            {{ item.name }} <br />
            <p class="text-sm">{{ item.time }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style>
  .bg-image {
    background: linear-gradient(rgba(0, 0, 0, 0.7), rgba(0, 0, 0, 0.7)),
      url("../assets/background/sulthan-auliya-ITxKCcbj7zc-unsplash.jpg");
  }
</style>

<script>
  // .. is an alias to /src
  import moment from "moment";
  import axios from "axios";

  export default {
    name: "HomeView",
    data() {
      return {
        timeNow: "",
        adzan: require("@/assets/adzan/adzan.mp3"),
        subuh: require("@/assets/adzan/subuh.mp3"),
        dayName: "",
        locality: "",
        year: "",
        day: "",
        month: "",
        cities: [],
        jadwal: [],
        cityId: "",
        principalSubdivision: "",
      };
    },
    methods: {
      changePosition: function () {
        var geo = confirm("Check kembali lokasi saat ini");
        if (geo) {
          const findMyState = () => {
            const success = (position) => {
              const latitude = position.coords.latitude;
              const longitude = position.coords.longitude;
              const geoApiUrl = `https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=${latitude}&longitude=${longitude}&localityLanguage=id`;

              fetch(geoApiUrl)
                .then((res) => res.json())
                .then((data) => {
                  this.locality = data.locality;
                  this.principalSubdivision = data.principalSubdivision;
                });
            };
            const error = () => {
              alert("Unable to retrieve your location");
            };
            navigator.geolocation.getCurrentPosition(success, error);
          };

          findMyState();
        } else {
          alert("dibatalkan");
        }
      },
    },
    components: {},
    created() {},
    async mounted() {
      const endpoint = "https://jadwal-shalat-api.herokuapp.com/";
      this.year = moment().format("YYYY");
      this.month = moment().format("MM");
      this.day = moment().format("DD");
      axios
        .get(`${endpoint}cities`)
        .then(function (result) {
          this.cities = result.data.data.filter((search) => {
            return search.cityName == this.locality;
          });
          var cityId = this.cities.map((item) => item.cityId);
          this.cityId = JSON.parse(cityId);
        })
        .catch(function (err) {});

      try {
        let response = await axios.get(
          `${endpoint}daily?date=${this.year}-${this.month}-${this.day}&cityId=${this.cityId}`
        );
        this.jadwal = response.data.data.data;
        console.log(this.jadwal);
      } catch (err) {
        console.log(err);
      }
      // axios
      //   .get(
      //     `${endpoint}daily?date=${this.year}-${this.month}-${this.day}&cityId=${this.cityId}`
      //   )
      //   .then(function (res) {
      //     this.jadwal = res.data.data;
      //   })
      //   .catch(function (err) {});
      const findMyState = () => {
        const success = (position) => {
          const latitude = position.coords.latitude;
          const longitude = position.coords.longitude;
          const geoApiUrl = `https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=${latitude}&longitude=${longitude}&localityLanguage=id`;

          fetch(geoApiUrl)
            .then((res) => res.json())
            .then((data) => {
              this.locality = data.locality;
              this.principalSubdivision = data.principalSubdivision;
            });
        };
        const error = () => {
          alert("Unable to retrieve your location");
        };
        navigator.geolocation.getCurrentPosition(success, error);
      };

      findMyState();
      if (!("Notification" in window)) {
        alert("This browser does not support desktop notification");
      }

      // Let's check whether notification permissions have already been granted
      else if (Notification.permission === "granted") {
        // If it's okay let's create a notification
      }

      // Otherwise, we need to ask the user for permission
      else if (Notification.permission !== "denied") {
        Notification.requestPermission().then(function (permission) {
          // If the user accepts, let's create a notification
          if (permission === "granted") {
            // let notification = new Notification("Hi there!");
          }
        });
      }

      setInterval(() => {
        this.timeNow = moment().format("HH:mm:ss");
        if (this.timeNow == "04:45:00") {
          let notification = new Notification("Sekarang Adzan Shubuh");
          var audio = new Audio(this.subuh);
          audio.play();
          setTimeout(function () {
            let notification = new Notification(
              "Selamat menunaikan ibadah sholat subuh"
            );
          }, 180000000);
        }
        if (this.timeNow == "12:01:00") {
          let notification = new Notification("Sekarang Adzan Dzuhur");
          var adzan = new Audio(this.adzan);
          adzan.play();
          setTimeout(function () {
            let notification = new Notification(
              "Selamat menunaikan ibadah sholat Dzuhur"
            );
          }, 180000000);
        }
        if (this.timeNow == "15:23:00") {
          let notification = new Notification("Sekarang Adzan Ashr");
          var adzan = new Audio(this.adzan);
          adzan.play();
          setTimeout(function () {
            let notification = new Notification(
              "Selamat menunaikan ibadah sholat Ashr"
            );
          }, 180000000);
        }
        if (this.timeNow == "17:55:00") {
          let notification = new Notification("Sekarang Adzan Maghrib");
          var adzan = new Audio(this.adzan);
          adzan.play();
          setTimeout(function () {
            let notification = new Notification(
              "Selamat menunaikan ibadah sholat Maghrib"
            );
          }, 180000000);
        }
        if (this.timeNow == "19:09:00") {
          let notification = new Notification("Sekarang Adzan Isya");
          var adzan = new Audio(this.adzan);
          adzan.play();
          setTimeout(function () {
            let notification = new Notification(
              "Selamat menunaikan ibadah sholat Isya"
            );
          }, 180000000);
        }
        this.dayName = moment().lang("id").format("ll");
      }, 1000);
    },
  };
</script>
