<template>
  <div id="app">
    <span id="alertBox">Скопировано</span>
    <!-- <nav-bar class="navComp" /> -->

    <section class="container">
      <div class="speedTest">
        <div class="speedTest__left">
          <div class="speedTest__left-speedShow">
            <p class="speed">{{ text }}</p>
            <p class="bytes">{{ bytes }}</p>
          </div>
          <button v-show="checkBtn" class="speedBtn" @click="checkSpeed()">
            Измерить
          </button>
          <button v-show="checkBtnNone" class="speedBtnHidden"></button>
        </div>
        <div class="speedTest__right">
          <div class="userInfo">
            <div class="listItem">
              <div class="listItem__left">Регион сайта:</div>
              <div class="listItem__right">{{ siteRegion }}</div>
            </div>
            <div class="listItem">
              <div class="listItem__left">Регион клиента:</div>
              <div class="listItem__right">{{ clientRegion }}</div>
            </div>
            <div class="listItem">
              <div class="listItem__left">Сервер:</div>
              <div class="listItem__right">91.234.218.52</div>
            </div>
            <div class="listItem">
              <div class="listItem__left">Клиент:</div>
              <div
                @click="copyClientIp('clientIp')"
                class="listItem__right clientIp"
              >
                <span id="clientIp">{{ clientIp }}</span>
                <img class="copyImg" src="/cpbt.png" alt="" />
              </div>
            </div>
            <div class="listItem">
              <div class="listItem__left">Средняя скорость:</div>
              <div class="listItem__right">{{ average }}</div>
            </div>
            <div class="listItem">
              <div class="listItem__left">Версия браузера:</div>
              <div class="listItem__right browserVersion">
                {{ browserVersion }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
// import NavBar from "./components/NavBar.vue";
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
Vue.use(VueAxios, axios);
export default {
  name: "App",
  components: {
    // NavBar,
  },
  data() {
    return {
      speed: 0,
      bytes: "",
      clientIp: null,
      serverIp: null,
      siteRegion: null,
      clientRegion: null,
      ipInfo: null,
      finished: false,
      checkBtn: true,
      checkBtnNone: false,
      average: "...",
      browserVersion: "...",
      source: axios.CancelToken.source(),
      text: "...",
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
      let time, loaded;
      const speeds = [];

      function speedToText(bytesPerMillisecond) {
        const megabitsPerSecond = bytesPerMillisecond / 125;
        if (megabitsPerSecond >= 1)
          return [`${Math.round(megabitsPerSecond)}`, "Мбит/с"];
        return [`${Math.round(bytesPerMillisecond * 8)}`, "Кбит/с"];
      }

      axios
        .get("https://st17.allmovies.uz/test.bin", {
          cancelToken: this.source.token,
          responseType: "blob",
          onDownloadProgress: (progressEvent) => {
            this.checkBtn = false;
            this.checkBtnNone = true;
            if (typeof time !== "number") time = progressEvent.timeStamp;
            if (typeof loaded !== "number") loaded = progressEvent.loaded;

            const timeDelta = progressEvent.timeStamp - time;
            const loadedDelta = progressEvent.loaded - loaded;

            if (timeDelta >= 1000) {
              const bytesPerMillisecond = loadedDelta / timeDelta;
              const [text, bytes] = speedToText(bytesPerMillisecond);
              this.text = text;
              this.bytes = bytes;
              speeds.push(bytesPerMillisecond);

              if (speeds.length >= 10) {
                this.average =
                  speeds.reduce((a, b) => a + b, 0) / speeds.length;
                this.average = speedToText(this.average);
                this.source.cancel("Test finished");
                this.checkBtn = true;
                this.checkBtnNone = false;
              }

              time += timeDelta;
              loaded += loadedDelta;
            }
          },
        })
        .catch((error) => {
          if (axios.isCancel(error)) {
            console.log("Request canceled", error.message);
          } else {
            this.$logError(error);
          }
        });
    },

    // checkSpeed() {
    //   let xhr = new XMLHttpRequest();
    //   xhr.responseType = "arraybuffer";
    //   xhr.onprogress = (data) => {
    //     const bytes = data.loaded - this.loaded;
    //     const time = data.timeStamp - this.timestamp;
    //     this.speed = bytes / time / 1024;
    //     if (this.speed <= 10) {
    //       this.speed = this.speed.toFixed(2);
    //     } else if (this.speed > 10) {
    //       this.speed = Math.round(bytes / time / 1024);
    //     }
    //     this.loaded = data.loaded;
    //     this.timestamp = data.timeStamp;
    //
    //   };
    //   xhr.open("GET", "https://st17.allmovies.uz/test.bin");
    //   xhr.send();
    //   setTimeout(() => {
    //     xhr.abort();
    //     this.finished = true;
    //
    //   }, 7000);
    // },
  },

  mounted() {
    axios.get("https://api.allplay.uz/api/v1/region").then((response) => {
      this.clientIp = response.data.data.ip;
      this.siteRegion = response.data.data.domain_region;
      this.clientRegion = response.data.data.ip_region;
    });
    this.browserVersion = navigator.userAgent;
  },
};
</script>

<style lang="scss">
@font-face {
  font-family: "LitteraText";
  src: local("Littera"),
    url(./assets/fonts/LitteraTextBook.ttf) format("truetype");
}

* {
  font-family: "LitteraText";
  color: #fff;
  box-sizing: border-box;
}
html {
  background: #000;
}
#app {
  display: flex;
  flex-direction: column;
}
.container {
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  padding: 0 30px;

  @media only screen and (max-width: 420px) {
    max-width: 420px;
    padding-left: 16px;
    padding-right: 16px;
    width: 100%;
  }
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

.speedTest {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  height: 70vh;
  @media only screen and (max-width: 800px) {
    flex-direction: column;
  }
  @media only screen and (max-width: 420px) {
    flex-direction: column;
  }

  &__left {
    display: flex;
    flex-direction: column;
    @media only screen and (max-width: 800px) {
      margin-bottom: 30px;
    }
    @media only screen and (max-width: 420px) {
      margin-bottom: 30px;
    }

    &-speedShow {
      border: 4px solid rgb(255, 153, 0);
      border-radius: 50%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      width: 300px;
      height: 300px;

      @media only screen and (max-width: 420px) {
        width: 200px;
        height: 200px;
      }

      .speed {
        font-size: 74px;
        margin: 0;

        @media only screen and (max-width: 420px) {
          font-size: 44px;
        }
      }
      .bytes {
        font-size: 22px;
        margin: 0;
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
    width: 50%;

    @media only screen and (max-width: 800px) {
      width: unset;
    }

    .userInfo {
      .listItem {
        display: flex;
        align-items: center;
        border-bottom: 1px solid #313134;
        font-size: 22px;
        padding: 24px 0;
        justify-content: space-between;
        @media only screen and (max-width: 420px) {
          font-size: 18px;
          padding: 16px 0;
        }
        &__left {
          color: #a4a4a5;
        }
      }
      .copyImg {
        height: 20px;
        padding-left: 5px;
        margin-bottom: -2px;
      }
      .clientIp {
        cursor: pointer;
      }
      .browserVersion {
        font-size: 14px;
        padding-left: 120px;
        text-align: right;
      }
    }
  }

  .speedBtn {
    font-size: 22px;
    background-color: #2b2b2b;
    padding: 9px;

    margin-top: 20px;
    border: 3px solid transparent;
    cursor: pointer;
    @media only screen and (max-width: 420px) {
      cursor: none;
      font-size: 18px;
    }

    &:hover {
      border: 3px solid rgb(255, 153, 0);
      @media only screen and (max-width: 420px) {
        border: transparent;
      }
    }
  }
  .speedBtnHidden {
    width: 200px;
    height: 48px;
    opacity: 0;
    margin-top: 20px;
  }
}
</style>
