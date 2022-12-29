vue
<template>
  <span>
    {{ text }}
  </span>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "SpeedTest",
  data() {
    return {
      text: "Подготовка...",
      source: this.$axios.CancelToken.source(),
    };
  },
  mounted() {
    let time: number, loaded: number;
    const speeds: number[] = [];

    function speedToText(bytesPerMillisecond: number) {
      const megabitsPerSecond = bytesPerMillisecond / 125;
      if (megabitsPerSecond >= 1)
        return `${Math.round(megabitsPerSecond)} Мбит/с`;
      return `${Math.round(bytesPerMillisecond * 8)} Кбит/с`;
    }

    this.$axios
      .get("https://st17.allmovies.uz/test.bin", {
        cancelToken: this.source.token,
        responseType: "blob",
        onDownloadProgress: (progressEvent) => {
          if (typeof time !== "number") time = progressEvent.timeStamp;
          if (typeof loaded !== "number") loaded = progressEvent.loaded;

          const timeDelta = progressEvent.timeStamp - time;
          const loadedDelta = progressEvent.loaded - loaded;

          if (timeDelta >= 1000) {
            const bytesPerMillisecond = loadedDelta / timeDelta;
            this.text = speedToText(bytesPerMillisecond);
            speeds.push(bytesPerMillisecond);

            if (speeds.length >= 10) {
              const average = speeds.reduce((a, b) => a + b, 0) / speeds.length;
              this.text += ` (сред. ${speedToText(average)})`;
              return this.source.cancel("Test finished");
            }

            time += timeDelta;
            loaded += loadedDelta;
          }
        },
      })
      .catch((error) => {
        if (this.$axios.isCancel(error)) {
          this.$log("Request canceled", error.message);
        } else {
          this.$logError(error);
        }
      });
  },
  beforeDestroy() {
    this.source.cancel("Component destroyed");
  },
});
</script>
