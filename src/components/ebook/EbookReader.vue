<template>
  <div class="ebook-reader">
    <!-- {{$route.params.fileName}} -->
    <div id="read"></div>
    <div class="ebook-reader-mask" @click="onMaskClick" @touchmove="move" @touchend="moveEnd"></div>
  </div>
</template>

<script>
// import { mapGetters } from "vuex";
// import { mapActions } from "vuex";
import { ebookMixin } from "../../utils/mixin";
import Epub from "epubjs";
import {
  getFontFamily,
  saveFontFamily,
  getFontSize,
  saveFontSize,
  getTheme,
  saveTheme,
  getLocation
} from "../../utils/localStorage";
import { addCss, flatten } from "../../utils/book";
global.Epub = Epub;
export default {
  mixins: [ebookMixin],
  data() {
    return {
      //   rendition: ""
    };
  },
  //   computed: {
  //     ...mapGetters(["fileName","menuVisible"])
  //   },
  methods: {
    move(e){
      let offsetY = 0
      if (this.firstOffsetY) {
        offsetY = e.changedTouches[0].clientY - this.firstOffsetY
        this.setOffsetY(offsetY)
      } else {
        this.firstOffsetY = e.changedTouches[0].clientY
      }
      e.preventDefault();
      e.stopPropagation()
    },
    moveEnd(e){
      this.setOffsetY(0)
      this.firstOffsetY = null
    },
    onMaskClick(e){
      const offsetX = e.offsetX
      const width = window.innerWidth
      if (offsetX>0 && offsetX<width*0.3) {
        this.prevPage()
      } else if (offsetX>0 && offsetX>width*0.7) {
        this.nextPage()
      } else {
        this.toggleTitleAndMenu()
      }
    },
    // ...mapActions(["setMenuVisible"]),
    prevPage() {
      if (this.rendition) {
        this.rendition.prev().then(() => {
          this.refreshLocation();
        });
        this.hideTitleAndMenu();
      }
    },
    nextPage() {
      if (this.rendition) {
        this.rendition.next().then(() => {
          this.refreshLocation();
        });
        this.hideTitleAndMenu();
      }
    },
    toggleTitleAndMenu() {
      //   this.$store.dispatch("setMenuVisible", !this.menuVisible);
      if (this.menuVisible) {
        this.setSettingVisible(-1);
      }
      this.setMenuVisible(!this.menuVisible);
      this.setFontFamilyVisible(false);
    },
    initFontSize() {
      let fontSize = getFontSize(this.fileName);
      if (!fontSize) {
        saveFontSize(this.fileName, this.defaultFontSize);
      } else {
        this.rendition.themes.fontSize(fontSize);
        this.setDefaultFontSize(fontSize);
      }
    },
    initFontFamily() {
      let font = getFontFamily(this.fileName);
      if (!font) {
        saveFontFamily(this.fileName, this.defaultFontFamily);
      } else {
        this.rendition.themes.font(font);
        this.setDefaultFontFamily(font);
      }
    },
    initTheme() {
      let defaultTheme = getTheme(this.fileName);
      if (!defaultTheme) {
        defaultTheme = this.themeList[0].name;
        saveTheme(this.fileName, defaultTheme);
      }
      this.setDefaultTheme(defaultTheme);
      this.themeList.forEach(theme => {
        this.rendition.themes.register(theme.name, theme.style);
      });
      this.rendition.themes.select(defaultTheme);
    },
    initRendition() {
      this.rendition = this.book.renderTo("read", {
        width: innerWidth,
        height: innerHeight,
        methods: "default"
      });
      const location  = getLocation(this.fileName)
      this.display(location, () => {
        this.initTheme();
        this.initFontSize();
        this.initFontFamily();
        this.initGlobalStyle();
      });
      // this.rendition.display().then(() => {
      //   this.initTheme();
      //   this.initFontSize();
      //   this.initFontFamily();
      //   this.initGlobalStyle();
      //   this.refreshLocation();
      // });
      this.rendition.hooks.content.register(contents => {
        Promise.all([
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/cabin.css`
          ),
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`
          ),
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`
          ),
          contents.addStylesheet(
            `${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`
          )
        ]).then(() => {});
        console.log(process.env.VUE_APP_RES_URL);
      });
    },
    initGesture() {
      this.rendition.on("touchstart", event => {
        this.touchStartX = event.changedTouches[0].clientX;
        this.touchStartTime = event.timeStamp;
      });
      this.rendition.on("touchend", event => {
        const offsetX = event.changedTouches[0].clientX - this.touchStartX;
        const time = event.timeStamp - this.touchStartTime;
        console.log(offsetX, time);
        if (time < 500 && offsetX > 40) {
          this.prevPage();
        } else if (time < 500 && offsetX < -40) {
          this.nextPage();
        } else {
          this.toggleTitleAndMenu();
        }
        // event.preventDefault();
        event.stopPropagation();
      });
    },
    parseBook(){
      this.book.loaded.cover.then(cover=>{
        // this.book.archive
        this.book.archive.createUrl(cover).then(url=>{
          this.setCover(url)
        })
        this.book.loaded.metadata.then(metadata=>{
          this.setMetadata(metadata)
        })
        this.book.loaded.navigation.then(nav=>{
          // console.log(flatten(nav.toc))
          const navItem = flatten(nav.toc)
          function find(item,level=0) {
            return !item.parent?level:find(navItem.filter(parentItem=>parentItem.id===item.parent)[0],++level)
          }
          navItem.forEach(item=>{
            item.level = find(item)
          })
          this.setNavigation(navItem)
        })
        // console.log(this.book.loaded.navigation)
      })
    },
    initEpub() {
      const url =
        process.env.VUE_APP_RES_URL + "/epub/" + this.fileName + ".epub";
      this.book = new Epub(url);
      this.setCurrentBook(this.book);
      this.initRendition();
      this.initGesture();
      this.parseBook()
      this.book.ready.then(() => {
        return this.book.locations
          .generate(
            750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16)
          )
          .then(locations => {
            // console.log(locations)
            this.setBookAvailable(true);
            this.refreshLocation()
          });
      });
    }
  },
  mounted() {
    // const fileName = ;
    this.setFileName(this.$route.params.fileName.split("|").join("/")).then(
      () => {
        this.initEpub();
      }
    );
    // console.log(`${baseUrl}${fileName}.epub`);
  }
};
</script>

<style scoped lang="scss">
@import "../../assets/styles/global.scss";
.ebook-reader {
  width: 100%;
  height: 100%;
  overflow: hidden;
  .ebook-reader-mask {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 150;
    top: 0;
    left: 0;
    // background: red;
  }
}
</style>