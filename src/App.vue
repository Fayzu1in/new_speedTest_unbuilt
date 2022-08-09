<template>
  <div id="app">
    <span id="alertBox">Скопировано</span>
    <nav-bar class="navComp" />
    <speed-test />
    <div id="stars"></div>
    <div id="stars2"></div>
    <div id="stars3"></div>
    <section class="speedTest container">
      <div class="speedTest__left">
        <div class="speedTest__left-region ipSection">
          <div class="info">
            <p class="info__text">Регион сайта:</p>

            <p>{{ siteRegion }}</p>
          </div>
          <div class="info">
            <p class="info__text">Регион клиента:</p>
            <p>{{ clientRegion }}</p>
          </div>
        </div>
      </div>
      <div class="speedTest__center">
        <div class="speedTest__center-speedShow">
          <p class="speed">{{ speed }}</p>
          <p class="bytes">{{ bytes }}</p>
        </div>
        <button
          v-show="checkBtn"
          class="speedTest__center-speedBtn"
          @click="checkSpeed()"
        >
          Начать
        </button>
        <button v-show="checkBtnNone" class="speedBtn">Start</button>
      </div>
      <div class="speedTest__right">
        <div class="speedTest__right-ip ipSection">
          <div class="info">
            <p class="info__text">Сервер:</p>
            <p>91.234.218.52</p>
          </div>
          <div @click="copyClientIp('clientIp')" class="info clientInfo">
            <p class="info__clientIp info__text">Клиент:</p>
            <p class="info__clientIp" id="clientIp">{{ clientIp }}</p>
          </div>
        </div>
        <div class="speedTest__right-region">
          <div>
            <p></p>
            <p></p>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import SpeedTest from "./page/SpeedTest.vue";
import NavBar from "./components/NavBar.vue";
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
Vue.use(VueAxios, axios);
export default {
  name: "App",
  components: {
    SpeedTest,
    NavBar,
  },
  data() {
    return {
      speed: 0,
      bytes: "MB / S",
      clientIp: null,
      serverIp: null,
      siteRegion: null,
      clientRegion: null,
      ipInfo: null,
      finished: false,
      checkBtn: true,
      checkBtnNone: false,
    };
  },
  methods: {
    // Кнопка копировании айпи адресса клиента
    copyClientIp(elementId) {
      let aux = document.createElement("input");
      aux.setAttribute("value", document.getElementById(elementId).innerHTML);
      document.body.appendChild(aux);
      aux.select();
      document.execCommand("copy");
      document.body.removeChild(aux);
      document.getElementById("alertBox").style.opacity = "1";
      setTimeout(function () {
        document.getElementById("alertBox").style.opacity = "0";
      }, 2000);
    },
    // Измееритель скорости интернета при скачивании файла типа .bin
    checkSpeed() {
      let xhr = new XMLHttpRequest();
      xhr.responseType = "arraybuffer";
      xhr.onprogress = (data) => {
        const bytes = data.loaded - this.loaded;
        const time = data.timeStamp - this.timestamp;
        // this.speed = Math.round(bytes / time / 1024);

        this.speed = bytes / time / 1024;
        if (this.speed <= 10) {
          this.speed = this.speed.toFixed(2);
        } else if (this.speed > 10) {
          this.speed = Math.round(bytes / time / 1024);
        }

        this.loaded = data.loaded;
        this.timestamp = data.timeStamp;
        this.checkBtn = false;
        this.checkBtnNone = true;
      };
      xhr.open("GET", "https://st16.allmovies.uz/test.bin");

      xhr.send();
      setTimeout(() => {
        xhr.abort();
        this.finished = true;
        this.checkBtn = true;
        this.checkBtnNone = false;
      }, 7000);
    },
  },
  mounted() {
    // Запрос на получение инфы о местопололожении абонента
    axios.get("https://api.allplay.uz/api/v1/region").then((response) => {
      this.clientIp = response.data.data.ip;
      this.siteRegion = response.data.data.domain_region;
      this.clientRegion = response.data.data.ip_region;
    });
  },
};
</script>

<style lang="scss">
@font-face {
  font-family: "LitteraText";
  src: local("Littera"),
    url(./assets/fonts/LitteraTextBook.ttf) format("truetype");
}

@import "./style/stars.css";
* {
  font-family: "LitteraText";
  color: #fff;
  box-sizing: border-box;
}
#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}
.container {
  max-width: 1200px;
  margin: 0 auto;
}
#alertBox {
  color: #fff;
  font-size: 20px;
  letter-spacing: -0.4px;
  padding: 12px;
  text-align: left;
  position: fixed;
  right: 0;
  top: 84px;
  background: rgba(50, 50, 50, 0.5);
  border-left: 0.5rem solid #2fb14b;
  backdrop-filter: blur(24px);
  transition: all 0.5s;
  padding: 0.75rem;
  margin-bottom: 0.5 rem;
  width: 300px;
  margin-right: 30px;
  opacity: 0;
}
.navComp {
  background-color: #000;
}
.speedTest {
  display: flex;
  justify-content: space-around;
  align-items: center;
  flex: 1;
  width: 100%;
  .ipSection {
    display: flex;
    flex-direction: column;

    background-color: #000;
    padding: 12px 32px;
    box-shadow: rgba(255, 255, 255, 0.1) 0px 1px 1px 0px inset,
      rgba(50, 50, 93, 0.25) 0px 50px 100px -20px,
      rgba(0, 0, 0, 0.3) 0px 30px 60px -30px;
    width: 100%;

    line-height: 32px;
    font-size: 22px;
    letter-spacing: -0.4px;
    .clientInfo {
      border-bottom: 0.5px solid rgba(255, 255, 255, 0.6);
    }
    .info {
      display: flex;
      justify-content: space-between;
      &__text {
        padding-right: 30px;
        color: rgba(255, 255, 255, 0.6);
      }
      &__clientIp {
        cursor: pointer;
      }
    }
  }

  &__left {
  }
  &__center {
    display: flex;
    flex-direction: column;
    align-items: center;
    &-speedBtn {
      background-color: rgb(255, 153, 0);
      padding: 12px 32px;
      box-shadow: rgba(255, 255, 255, 0.1) 0px 1px 1px 0px inset,
        rgba(50, 50, 93, 0.25) 0px 50px 100px -20px,
        rgba(0, 0, 0, 0.3) 0px 30px 60px -30px;

      align-items: inherit;
      border: none;
      font-size: 22px;
      margin-top: 20px;
      cursor: pointer;
      letter-spacing: -0.4px;
    }
    .speedBtn {
      background-color: rgb(255, 153, 0);
      padding: 12px 32px;
      opacity: 0;
      align-items: inherit;
      border: none;
      font-size: 22px;
      margin-top: 20px;
      letter-spacing: -0.4px;
    }

    &-speedShow {
      border: 3px solid rgb(255, 153, 0);
      border-radius: 50%;
      // padding: 40px 60px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      width: 200px;
      height: 200px;
      .speed {
        font-size: 80px;
      }
      .bytes {
        font-size: 30px;
      }

      animation: crescendo 1.5s alternate infinite ease-in;
      @keyframes crescendo {
        0% {
          transform: scale(1);
        }
        100% {
          transform: scale(0.9);
        }
      }
    }
  }
  &__right {
  }
}
</style>
