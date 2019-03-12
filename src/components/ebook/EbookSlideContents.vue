<template>
  <div class="ebook-slide-contents">
    <div class="slide-contents-search-wrapper">
      <div class="slide-contents-search-input-wrapper">
        <div class="sliede-content-search-icon">
          <span class="icon-search"></span>
        </div>
        <input
          type="text"
          class="slide-contents-search-input"
          :placeholder="$t('book.searchHint')"
          @click="showSearchPage()"
        >
      </div>
      <div
        class="slide-contents-search-cancel"
        v-if="searchVisible"
        @click="hideSerachPage()"
      >{{$t('book.cancel')}}</div>
    </div>
    <div class="slide-contents-book-wrapper">
      <div class="slide-contents-book-img-wrapper">
        <img :src="cover" alt class="slide-contents-book-img">
      </div>
      <div class="slide-contents-book-info-wrapper">
        <div class="slide-contents-book-title">{{metadata.title}}</div>
        <div class="slide-contents-book-author">{{metadata.creator}}</div>
      </div>
      <div class="slide-contents-book-progress-wrapper">
        <div class="slide-contents-book-progress">
          <span class="progress">{{progress+'%'}}</span>
          <span class="progress-text">{{$t('book.haveRead2')}}</span>
        </div>
        <div class="slide-contents-book-time">{{getReadTimeText()}}</div>
      </div>
    </div>
  </div>
</template>

<script>
import { ebookMixin } from "../../utils/mixin";
export default {
  mixins: [ebookMixin],
  data() {
    return {
      searchVisible: false
    };
  },
  methods: {
    showSearchPage() {
      this.searchVisible = true;
    },
    hideSerachPage() {
      this.searchVisible = false;
    }
  }
};
</script>

<style scoped lang="scss">
@import "../../assets/styles/global.scss";
.ebook-slide-contents {
  width: 100%;
  .slide-contents-search-wrapper {
    display: flex;
    width: 100%;
    height: px2rem(36);
    margin: px2rem(20) 0 px2rem(10) 0;
    padding: 0 px2rem(15);
    box-sizing: border-box;

    .slide-contents-search-input-wrapper {
      flex: 1;
      @include center;
      .sliede-content-search-icon {
        font-size: px2rem(12);
        flex: 0 0 px2rem(28);
        @include center;
      }

      .slide-contents-search-input {
        flex: 1;
        width: 100%;
        height: px2rem(32);
        font-size: px2rem(14);
        background: transparent;
        border: none;
        outline: none;
      }
    }
    .slide-contents-search-cancel {
      flex: 0 0 px2rem(50);
      font-size: px2rem(14);
      @include right;
    }
  }
  .slide-contents-book-wrapper {
      font-size: 0;
    display: flex;
    width: 100%;
    height: px2rem(90);
    padding:px2rem(10) px2rem(15) px2rem(20) px2rem(15);
    box-sizing: border-box;
    .slide-contents-book-img-wrapper {
      flex: 0 0 px2rem(45);
      .slide-contents-book-img {
        width: px2rem(45);
        height: px2rem(60);
      }
    }
    .slide-contents-book-info-wrapper {
      flex: 1;
      padding: 0 px2rem(10);
    box-sizing: border-box;
      .slide-contents-book-title {
          width: px2rem(153.75);
        font-size: px2rem(14);
        @include ellipsis2(3);
      }
      .slide-contents-book-author {
          width: px2rem(153.75);
        font-size: px2rem(12);
        @include ellipsis;
        margin-top: px2rem(5);
      }
    }
    .slide-contents-book-progress-wrapper {
      .slide-contents-book-progress {
        .progress {
          font-size: px2rem(14);
        }
        .progress-text {
          font-size: px2rem(12);
        }
      }
      .slide-contents-book-time {
          font-size: px2rem(12);
          margin-top: px2rem(5);
      }
    }
  }
}
</style>