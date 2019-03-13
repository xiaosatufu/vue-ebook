<template>
  <div class="ebook-bookmark" ref="bookmark">
    <div class="ebook-bookmark-text-wrapper">
      <div class="ebook-bookmark-down-wrapper" ref="iconDown">
        <span class="icon-down"></span>
      </div>
      <div class="ebook-bookmark-text">{{text}}</div>
    </div>
    <div class="ebook-bookmark-icon-wrapper" :style="isFixed?fixedStyle:{}">
      <!-- <div class="icon"></div> -->
      <bookmark :color="color" :height="35" :width="15"></bookmark>
    </div>
  </div>
</template>

<script>
import Bookmark from "../common/Bookmark";
import { realPx } from "../../utils/utils";
import { ebookMixin } from "../../utils/mixin.js";
import {getBookmark , setBookmark, saveBookmark} from "../../utils/localStorage.js"
const BLUE = "#346CBC";
const WHITE = "#fff";
export default {
  mixins: [ebookMixin],
  components: {
    Bookmark
  },
  computed: {
    height() {
      return realPx(35);
    },
    threshold() {
      return realPx(55);
    },
    fixedStyle() {
      return {
        position: "fixed",
        top: 0,
        right: `${(window.innerWidth - this.$refs.bookmark.clientWidth) / 2}px`
      };
    }
  },
  watch: {
    offsetY(v) {
      if (!this.bookAvailable || this.menuVisible || this.settingVisible >= 0) {
        return;
      }
      if (v >= this.height && v < this.threshold) {
        this.beforeThreshold(v);
      } else if (v >= this.threshold) {
        this.afterThreshold(v);
      } else if (v > 0 && v < this.height) {
        this.beforeHeight();
      } else if (v === 0) {
        this.restore();
      }
    },
    isBookmark(isBookmark){
        if (isBookmark) {
            this.color = BLUE
            this.isFixed = true
        } else {
            this.color = WHITE
            this.isFixed = false
        }
    }
  },
  data() {
    return {
      //   text: this.$t('book.pulldownAddMark'),
      text: "",
      color: WHITE,
      isFixed: false
    };
  },
  methods: {
    addBookMark() {
        this.bookmark = getBookmark(this.fileName)
        if (!this.bookmark) {
            this.bookmark = []
        }
        const currentLocation = this.currentBook.rendition.currentLocation()
        const cfibase = currentLocation.start.cfi.replace(/!.*/,'')
        const cfiStart = currentLocation.start.cfi.replace(/.*!/,'').replace(/\)$/,'')
        const cfiEnd = currentLocation.end.cfi.replace(/.*!/,'').replace(/\)$/,'')
        const cfiRange = `${cfibase}!,${cfiStart},${cfiEnd})`
        this.currentBook.getRange(cfiRange).then(range=>{
            const text = range.toString().replace(/\s\s/g,'')
            this.bookmark.push({
                cfi:currentLocation.start.cfi,
                text:text
            })
            saveBookmark(this.fileName,this.bookmark)
        })
    },
    removeBookmark() {
        const currentLocation = this.currentBook.rendition.currentLocation()
        const cfi = currentLocation.start.cfi
        if (this.bookmark) {
        // this.bookmark = getBookmark(this.fileName)
            // this.bookmark = this.bookmark.filter(item=>item.cfi != cfi)
            saveBookmark(this.fileName,this.bookmark.filter(item=>item.cfi != cfi))
            this.setIsBookmark(false)
        }
    },
    restore() {
      setTimeout(() => {
        this.$refs.bookmark.style.top = `${-this.height}px`;
        this.$refs.iconDown.style.transform = "rotate(0deg)";
      }, 200);
      if (this.isFixed) {
        this.setIsBookmark(true);
        this.addBookMark();
      } else {
        this.setIsBookmark(false);
        this.removeBookmark();
      }
    },
    beforeHeight() {
      if (this.isBookmark) {
        //   this.text = this.$t('book.')
        this.text = this.$t("book.pulldownDeleteMark");
        this.color = BLUE;
        this.isFixed = true;
      } else {
        this.text = this.$t("book.pulldownAddMark");
        this.color = WHITE;
        this.isFixed = false;
      }
    },
    beforeThreshold(v) {
      this.$refs.bookmark.style.top = `${-v}px`;
      this.beforeHeight();
      //   if (this.isBookmark) {
      //     //   this.text = this.$t('book.')
      //     this.text = this.$t("book.pulldownDeleteMark");
      //     this.color = WHITE;
      //   } else {
      //     this.text = this.$t("book.pulldownAddMark");
      //     this.color = WHITE;
      //   }
      const iconDown = this.$refs.iconDown;
      if (iconDown.style.transform === "rotate(180deg)") {
        iconDown.style.transform = "rotate(0deg)";
      }
    },
    afterThreshold(v) {
      this.$refs.bookmark.style.top = `${-v}px`;
      if (this.isBookmark) {
        this.text = this.$t("book.releaseDeleteMark");
        this.color = WHITE;
        this.isFixed = false;
      } else {
        this.text = this.$t("book.releaseAddMark");
        this.color = BLUE;
        this.isFixed = true;
      }
      const iconDown = this.$refs.iconDown;
      if (
        iconDown.style.transform === "" ||
        iconDown.style.transform === "rotate(0deg)"
      ) {
        iconDown.style.transform = "rotate(180deg)";
      }
    }
  }
};
</script>

<style scoped lang="scss">
@import "../../assets/styles/global.scss";
.ebook-bookmark {
  position: absolute;
  top: px2rem(-35);
  left: 0;
  z-index: 200;
  width: 100%;
  height: px2rem(35);
  .ebook-bookmark-text-wrapper {
    position: absolute;
    right: px2rem(45);
    bottom: 0;
    display: flex;
    .ebook-bookmark-down-wrapper {
      font-size: px2rem(14);
      color: #fff;
      transition: all 0.2s linear;
      @include center;
    }
    .ebook-bookmark-text {
      font-size: px2rem(14);
      color: #fff;
    }
  }
  .ebook-bookmark-icon-wrapper {
    position: absolute;
    right: 0;
    bottom: 0;
    margin-right: px2rem(15);
  }
}
</style>