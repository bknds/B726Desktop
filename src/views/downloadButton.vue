<template>
  <div class="wapper">
    <bknds-progress v-if="downloadInfo && downloadInfo.progress !== 100" color="#000" :value="downloadInfo.progress" size="18" />
    <a
      class="music-list-btn iconfont-download"
      href="javascript:;"
      @click="download()"
      v-if="!downloadInfo"
    ></a>
    <a
      class="music-list-btn iconfont-download-end"
      href="javascript:;"
      v-if="downloadInfo && downloadInfo.progress === 100"
    ></a>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed } from "vue";
import { useStore } from "vuex";
import useRequests from "@/plugins/api";
import usePlayerFn from "@/plugins/player";

export default defineComponent({
  props: {
    song: {
      type: Object,
      default: () => {},
    },
  },

  setup(props) {
    const $store = useStore();
    const { getMiGuMusicAudioUrlApi } = useRequests();
    const { getSingersName } = usePlayerFn();
    const song = computed(() => props.song);
    const downloader = computed(() => $store.state.downloader);

    const getDownloadInfo = () => {
      return downloader.value.downloads.filter(
        (item: any) => item.id === song.value.id
      );
    };
    const downloadInfo = computed(() => getDownloadInfo()[0]);

    const download = async () => {
      const audioUrl = await getMiGuMusicAudioUrlApi({
        id: song.value.id,
        name: song.value.songName,
        contentId: song.value.contentId,
        singer: getSingersName(song.value.singers),
      });
      $store.dispatch("downloader/download", {
        id: song.value.id,
        downloadUrl: audioUrl.data.url,
        filename: song.value.songName,
      });
    };

    return {
      download,
      downloadInfo,
    };
  },
});
</script>

<style lang="scss" scoped>
@import "@/assets/styles/theme/conf.scss";

.wapper {
  position: relative;
  box-sizing: border-box;
  width: 20px;
  height: 20px;

  .rect {
    position: absolute;
    width: 10px;
    height: 20px;
    overflow: hidden;

    .circle {
      position: absolute;
      width: 20px;
      height: 20px;
      border-radius: 50%;
      box-sizing: border-box;
      border: 3px solid indianred;

      &.circle_right {
        left: -10px;
        clip: rect(0px 10px 20px 0px);
        /* animation: roll 5s linear 0s 1 forwards; */
      }

      &.circle_left {
        clip: rect(0px 20px 20px 10px);
        /* animation: roll 5s linear 5s 1 forwards; */
      }
    }

    &.right {
      left: 10px;
    }
  }
}

.iconfont-download-end {
  color: $primary-color !important;
}

@keyframes roll {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(180deg);
  }
}
</style>
