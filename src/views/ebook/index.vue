<template>
  <div class="ebook" ref="ebook">
    <ebook-title></ebook-title>
    <ebook-reader></ebook-reader>
    <ebook-menu></ebook-menu>
    <ebook-bookmark></ebook-bookmark>
  </div>
</template>

<script>
import EbookReader from "../../components/ebook/EbookReader";
// import EbookTitle from '../../'
import EbookTitle from "../../components/ebook/EbookTitle";
import EbookMenu from "../../components/ebook/EbookMenu.vue";
import EbookBookmark from "../../components/ebook/EbookBookmark";
// import { setInterval } from 'timers';
import { saveReadTime, getReadTime } from "../../utils/localStorage";
import { ebookMixin } from "../../utils/mixin";
export default {
  mixins: [ebookMixin],
  components: {
    EbookReader,
    EbookTitle,
    EbookMenu,
    EbookBookmark
  },
  watch: {
    offsetY(v) {
      if (!this.menuVisible && this.bookAvailable) {
        if (v > 0) {
          this.move(v);
        } else if (v == 0) {
          this.restore();
        }
      }
    }
  },
  methods: {
    move(v) {
      this.$refs.ebook.style.top = v + "px";
    },
    restore() {
      this.$refs.ebook.style.top = 0 + "px";
      this.$refs.ebook.style.transition = "all .2s linear";
      setTimeout(() => {
        this.$refs.ebook.style.transition = "";
      }, 200);
    },
    startLoopReadTime() {
      let readTime = getReadTime(this.fileName);
      if (!readTime) {
        readTime = 0;
      }
      this.task = setInterval(() => {
        readTime++;
        if (readTime % 30 === 0) {
          saveReadTime(this.fileName, readTime);
        }
      }, 1000);
    }
  },
  mounted() {
    this.startLoopReadTime();
  },
  beforeDestroy() {
    if (this.task) {
      clearInterval(this.task);
    }
  }
};
</script>

<style scoped lang="scss">
@import "../../assets/styles/global.scss";
.ebook {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>