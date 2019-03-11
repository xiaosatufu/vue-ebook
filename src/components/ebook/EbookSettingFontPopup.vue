<template>
  <transition name="popup-slide-up">
    <div class="ebook-popup-list" v-show="fontFamilyVisible">
      <div class="ebook-popup-title">
        <div class="ebook-popup-title-icon" @click="hide">
          <span class="icon-down2"></span>
        </div>
        <div class="ebook-popup-title-text">选择字体</div>
      </div>
      <div class="ebook-popup-list-wrapper">
        <div class="ebook-popup-item" @click="setFontFamily(item.font)"  v-for="(item,index) in fontFamilyList" :key="index">
          <div class="ebook-popup-item-text" :class="{'selected':isSelected(item)}">{{item.font}}</div>
          <div class="ebook-popup-item-check" v-if="isSelected(item)">
            <span class="icon-check"></span>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import { ebookMixin } from "../../utils/mixin.js";
import { FONT_FAMILY } from "../../utils/book.js";
// import {setLocalStorage,getLocalStorage} from "../../utils/localStorage"
import {saveFontFamily} from "../../utils/localStorage"
export default {
  mixins: [ebookMixin],
  data() {
    return {
      fontFamilyList: FONT_FAMILY
    };
  },
  mounted () {
    // setLocalStorage(this.fileName,this.defaultFontFamily)  
  },
  methods: {
    hide() {
      this.setFontFamilyVisible(false);
    },
    isSelected(item) {
      return this.defaultFontFamily === item.font ? true : false;
    },
    setFontFamily(font){
        this.setDefaultFontFamily(font)
        saveFontFamily(this.fileName,font)
        if (font==='Default') {
        this.currentBook.rendition.themes.font('Times New Roman')
        } else {
        this.currentBook.rendition.themes.font(font)
        }
    }
  }
};
</script>

<style scoped lang="scss">
@import "../../assets/styles/global.scss";
.ebook-popup-list {
  position: absolute;
  bottom: 0;
  left: 0;
  z-index: 300;
  width: 100%;
  box-shadow: 0 px2rem(-4) px2rem(6) rgba(0, 0, 0, 0.1);
  background: #fff;
  .ebook-popup-title {
    padding: px2rem(15);
    box-sizing: border-box;
    border-bottom: solid #b8b9bb px2rem(1);
    text-align: center;
    @include center;
    position: relative;
    .ebook-popup-title-icon {
      position: absolute;
      left: px2rem(15);
      top: 0;
      height: 100%;
      font-size: px2rem(16);
      font-weight: bold;
      @include center;
    }
    .ebook-popup-title-text {
      font-size: px2rem(14);
      font-weight: 700;
    }
  }
  .ebook-popup-list-wrapper {
    .ebook-popup-item {
      display: flex;
      padding: px2rem(15);
      .ebook-popup-item-text {
        flex: 1;
        font-size: px2rem(14);
        text-align: left;
        &.selected {
          color: #346cb9;
          font-weight: bold;
        }
      }
      .ebook-popup-item-check {
        flex: 1;
        text-align: right;
        font-size: px2rem(14);
        font-weight: bold;
        color: #346cb9;
        &.selected {
          font-weight: bold;
        }
      }
    }
  }
}
</style>