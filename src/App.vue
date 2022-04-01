<template>
  <div class="container">
    123 123 123
    <app-modal 
      ref="modal" 
      fullscreen
      :hideCloseButton="!!adCloseTimerCount"
    >
      <template v-slot:header>
        <div 
          v-if="adCloseTimerCount && adCloseTimerCount !== 15000" 
          class="modal__timer"
          v-text="adCloseTimerText"
        />
      </template>

      <video-player
        ref="video-player"
        playsinline
        :vast="vast"
        @resume="removeModal"
        @close="removeModal"
        @loadAd="initAdTimer"
      />
    </app-modal>
  </div>
</template>

<script>

import VideoPlayer from './components/VideoPlayer';
import AppModal from './components/AppModal';

export default {
  components: {
    VideoPlayer,
    AppModal,
  },

  data: () => ({
    vast: 'https://yandex.ru/ads/adfox/168627/getCode?p1=bdcfe&p2=ekza&pfc=zymd&pfb=dkvcj',
    adCloseTimerCount: 15000,
  }),

  computed: {
    adCloseTimerText() {
      const adCloseTimer = Math.ceil(this.adCloseTimerCount/1000);
      return `Закрыть рекламу через ${adCloseTimer} сек`;
    },
  },

  mounted() {
    document.addEventListener('click', this.initAd);
  },

  methods: {
    initAd() {
      const modal = this.$refs.modal;
      const videoPlayer = this.$refs['video-player'];
      if('open' in modal) modal.open();
      if('loadAds' in videoPlayer) videoPlayer.loadAds();
      document.removeEventListener('click', this.initAd);
    },
    initAdTimer() {
      const timer = setInterval(() => {
        if(this.adCloseTimerCount > 0) {
          this.adCloseTimerCount -= 50;
        } else {
          clearInterval(timer);
        }
      }, 50);
    },
    removeModal() {
      const modal = this.$refs.modal.$el;
      if('remove' in modal) modal.remove();
      document.removeEventListener('click', this.initAd);
    },
  },
};
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}

.modal__timer {
  color: #fff;
  font-weight: bold;
}
</style>