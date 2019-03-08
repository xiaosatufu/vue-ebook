<template>
  <div class="ebook-reader">
    <!-- {{$route.params.fileName}} -->
    <div id="read"></div>
  </div>
</template>

<script>
// import { mapGetters } from "vuex";
// import { mapActions } from "vuex";
import { ebookMixin } from "../../utils/mixin";
import Epub from "epubjs";
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
    // ...mapActions(["setMenuVisible"]),
    prevPage() {
      if (this.rendition) {
        this.rendition.prev();
        this.hideTitleAndMenu();
      }
    },
    nextPage() {
      if (this.rendition) {
        this.rendition.next();
        this.hideTitleAndMenu();
      }
    },
    toggleTitleAndMenu() {
      //   this.$store.dispatch("setMenuVisible", !this.menuVisible);
      if (this.menuVisible) {
          this.setSettingVisible(-1)
      }
      this.setMenuVisible(!this.menuVisible);
      this.setFontFamilyVisible(false)
    },
    hideTitleAndMenu() {
      //   this.$store.dispatch("setMenuVisible", false);
      this.setMenuVisible(false);
      this.setSettingVisible(-1)
      this.setFontFamilyVisible(false)
    },
    initEpub() {
      const url = "http://192.168.2.121:8081/epub/" + this.fileName + ".epub";
      console.log(url);
      this.book = new Epub(url);
      this.setCurrentBook(this.book)
      console.log(this.book);
      this.rendition = this.book.renderTo("read", {
        width: innerWidth,
        height: innerHeight,
        methods: "default"
      });
      this.rendition.display();
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
      this.rendition.hooks.content.register(contents=>{
        Promise.all([
        contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/cabin.css`),
        contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`),
        contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`),
        contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`)
        ]).then(()=>{
          
        })
        console.log(process.env.VUE_APP_RES_URL)
      })
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

<style scoped>
</style>