<template>
  <div class="video__container">
    <video 
      ref="video"
      class="video__element"
      :playsinline="playsinline"
      @play.prevent="loadAds"
    />
    <div ref="ad" class="ad__container"/>
    <div v-if="!isAdsLoad" class="video__loader"/>
  </div>
</template>

<script>
let adsManager;
export default {
	name: 'VideoPlayer',

  props: {
    vast: String,
    playsinline: Boolean,
  },

  data: () => ({
    videoElement: null,
    adDisplayContainer: null,

    adsLoader: null,
    isAdsLoad: false,
    isAdsManagerLoad: false,
    isAdsPlaying: false,
  }),

  mounted() {
    if(!this.vast) this.$emit('close');
    this.videoElement = this.$refs.video;
    this.initAds();

    window.addEventListener('resize', () => {
      if(!adsManager) return;
      const width = this.videoElement.clientWidth;
      const height = this.videoElement.clientHeight;
      adsManager.resize(width, height, google.ima.ViewMode.NORMAL);
    });
  },

  methods: {
    initAds() {
      const adsContainer = this.$refs.ad;
      const adDisplayContainer = new google.ima.AdDisplayContainer(adsContainer, this.videoElement);
      const adsLoader = new google.ima.AdsLoader(adDisplayContainer);
      this.adDisplayContainer = adDisplayContainer;

      const { ADS_MANAGER_LOADED } = google.ima.AdsManagerLoadedEvent.Type;
      const { AD_ERROR } = google.ima.AdErrorEvent.Type;
      adsLoader.addEventListener(ADS_MANAGER_LOADED, this.onAdsManagerLoaded, false);
      adsLoader.addEventListener(AD_ERROR, this.onAdError, false);

      this.videoElement.addEventListener('ended', () => {
        adsLoader.contentComplete;
        this.$emit('close');
      });

      const adsRequest = new google.ima.AdsRequest();
      adsRequest.adTagUrl = this.vast;

      adsRequest.linearAdSlotWidth = this.videoElement.clientWidth;
      adsRequest.linearAdSlotHeight = this.videoElement.clientHeight;
      adsRequest.nonLinearAdSlotWidth = this.videoElement.clientWidth;
      adsRequest.nonLinearAdSlotHeight = this.videoElement.clientHeight / 3;

      adsLoader.requestAds(adsRequest);
    },

    onAdsManagerLoaded(event) {
      this.isAdsManagerLoad = true;

      const onContentResume = () => {
        this.isAdsPlaying = false;
        this.$emit('resume');
        this.videoElement.play();
      };

      try {
        adsManager = event.getAdsManager(this.videoElement);

        const { AD_ERROR } = google.ima.AdErrorEvent.Type;
        const { CONTENT_RESUME_REQUESTED, LOADED, COMPLETE } = google.ima.AdEvent.Type;
        adsManager.addEventListener(AD_ERROR, this.onAdError);
        adsManager.addEventListener(CONTENT_RESUME_REQUESTED, onContentResume);
        adsManager.addEventListener(COMPLETE, onContentResume);
        adsManager.addEventListener(LOADED, this.onAdLoaded);
      } catch(err) {
        onContentResume();
      }
    },

    onAdLoaded(event) {
      if(event.getAd().isLinear()) return;
      this.videoElement.play();
    },

    onAdError(event) {
      this.isAdsManagerLoad = true;
      this.isAdsPlaying = false;
      this.$emit('resume');
      if(!adsManager) return;
      adsManager.destroy();
    },

    loadAds() {
      if(!this.isAdsManagerLoad) return setTimeout(this.loadAds, 50);
      if(this.isAdsLoad) return;
      this.isAdsLoad = true;

      this.$emit('loadAd');
      this.videoElement.load();
      this.videoElement.pause();
      this.adDisplayContainer.initialize();

      try {
        const width = this.videoElement.clientWidth;
        const height = this.videoElement.clientHeight;
        adsManager.init(width, height, google.ima.ViewMode.NORMAL);
        adsManager.start();
        this.videoElement.addEventListener('click', () => this.$emit('resume'));
      } catch (err) {
        this.$emit('resume');
        this.videoElement.play();
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.video {
  &__container {
    position: relative;
  }
  &__element {
    width: 100%;
    height: 100%;
  }
  &__loader {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    &::after {
      content: "";
      width: 50px;
      height: 50px;
      outline: 4px solid #fff;
      border-radius: 10px;
      transition: 1s;
      animation: rotate 2s linear infinite;
    }
  }
}
.ad {
  &__container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
}

@keyframes rotate {
  to { 
    transform: rotate(360deg);
  }
}
</style>