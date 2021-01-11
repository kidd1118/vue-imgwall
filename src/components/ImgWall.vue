
<template>
  <div class="img-wall">
		<row v-for="row in state.rows" :key="row" v-bind:thumbnails="thumbnailsByRow(row)"></row>
		<popup ref="popup"></popup>
	</div>
</template>

<script>
import { reactive, computed, watch, onMounted } from 'vue/dist/vue.esm-bundler.js';

export default {
  setup(props) {
    const state = reactive({
      wallId: "img-wall_1234",
      maxRows: 3,
      maxCells: 5,
      cellWidth: "auto",
      cellHeight: "auto",
      rows: 0,
      cells: 0,
      layout: "Landscape",
      fixedLayout: false,

      thumbnails: computed(() => {
          //var _thumbnails = this.tournament ? this.tournament.thumbnails : [];

          //for testing
          var _thumbnails = [
            { thumbnailId: 5, name: "05" },
            { thumbnailId: 6, name: "06" },
            { thumbnailId: 1, name: "01" },
            { thumbnailId: 6, name: "06" },
            { thumbnailId: 7, name: "07" },
            { thumbnailId: 8, name: "08" },
            { thumbnailId: 6, name: "06" },
            { thumbnailId: 7, name: "07" },
            { thumbnailId: 8, name: "08" },
            { thumbnailId: 6, name: "06" },
            { thumbnailId: 8, name: "08" },
            { thumbnailId: 2, name: "02" },
            { thumbnailId: 3, name: "03" },
            { thumbnailId: 4, name: "04" },
            { thumbnailId: 5, name: "05" },
            { thumbnailId: 4, name: "04" }
          ];

          return _thumbnails;
        }
      )
    });
    /**
     * 清除並且重繪
     * */
    function redraw() {
      console.warn('============>Leo');
      state.rows = 0;
      state.cells = 0;
      var newLayout = checkLayout();
      if (newLayout.toLowerCase() === "landscape") {
        state.maxRows = 3;
        state.maxCells = 5;
      } else {
        state.maxCells = 5;
        state.maxRows = 3;
      }

      init();
      resize();
    }

    /**
     * box排列大小與規則
     */
    function init() {
      var totalThumbnails = state.thumbnails.length,
        minSqrt = isNaN(totalThumbnails)
          ? 0
          : Math.floor(Math.sqrt(totalThumbnails));

      if (totalThumbnails <= state.maxCells * state.maxRows) {
        if (state.layout == 'Landscape') {
          state.rows = Math.min(minSqrt, state.maxRows);
          state.cells = Math.ceil(totalThumbnails / state.rows);
        } else {
          state.cells = Math.min(minSqrt, state.maxCells);
          state.rows = Math.ceil(totalThumbnails / state.cells);
        }
      } else {
        state.cells = state.maxCells;
        state.rows = Math.ceil(totalThumbnails / state.cells);
      }
    }

    function resize() {
      // //選擇長或寬最小值去計算cell width
      // if (this.$el.offsetWidth > 0 && this.$el.offsetHeight > 0) {
      //   this.cellWidth =
      //     Math.floor(
      //       Math.min(
      //         this.$el.offsetHeight / Math.min(this.rows, this.maxRows),
      //         this.$el.offsetWidth / Math.min(this.cells, this.maxCells)
      //       )
      //     ) + "px";
      // } else {
      //   this.cellWidth = 100 / this.cells + "%";
      // }
      // // console.warn('============>Leo, cell width = ' + this.cellWidth);
      // // console.warn("this.$el.offsetWidth = " + this.$el.offsetWidth);
      // // console.warn("this.$el.offsetHeight = " + this.$el.offsetHeight);
    }

    function thumbnailsByRow(row) {
      return state.thumbnails.slice(
        (row - 1) * state.cells,
        Math.min(row * state.cells, state.thumbnails.length)
      );
    }

    function checkLayout() {
      if (state.fixedLayout) return;
      var mql = window.matchMedia("(orientation: portrait)");

      if (mql.matches) {
        state.layout = "Portrait";
      } else {
        state.layout = "Landscape";
      }
      return state.layout;
    }

    function getLaunchUrl(thumbnail) {
      // todo: get url
      return "?gamesn=" + thumbnail.thumbnailId;
    }

    onMounted(() => {
      //var self = this;

      redraw();

      // this.$on(
      //   "appResized",
      //   function() {
      //     //console.warn(">>img-wall resize...");
      //     this.checkLayout();
      //     this.resize();
      //   }.bind(this)
      // );

      // this.$on(
      //   "zoomIn",
      //   function(url, thumbnail) {
      //     this.$refs.popup.setImgSource(url);
      //     this.$refs.popup.setThumbnail(thumbnail);
      //     this.$refs.popup.show();
      //   }.bind(this)
      // );

      // this.$on(
      //   "launchUrl",
      //   function(thumbnail) {
      //     //window.open(this.getLaunchUrl(thumbnail));
      //     // CRD-695
      //     this.$emit("tryLaunchingGameByIcon", thumbnail);
      //   }.bind(this)
      // );
      /*
          觀察 mobile orientation 改變就重排 game icons
        */
      window.addEventListener("orientationchange", function() {
        setTimeout(function() {
          redraw();
        }, 200);
      });
    });
    watch({
      tournament: function() {
        //console.log("tournament @ " + this.wallId);
      },

      layout: function() {
        //todo: re-build if orientation is changed.
      }
    });
    return {
      props,
      state,
      redraw,
      init,
      resize,
      thumbnailsByRow,
      checkLayout,
      getLaunchUrl
    };
  },
  components: {
    popup: {
      setup(props, { emit }) {
        const state = reactive({
          isVisible: false,
          thumbnail: null,
          styles: {
            backgroundImage: ""
          }
        });
        function setImgSource(url) {
          state.styles.backgroundImage = "url(" + url + ")";
        }

        function show() {
          state.isVisible = true;
        }

        function onClick(event) {
          event.stopImmediatePropagation();
          //console.log(">>>click icon", this.thumbnail);
          emit("launchUrl", this.thumbnail); //notify img-wall instance
        }

        function onClickClose(event) {
          event.stopImmediatePropagation();
          state.isVisible = false;
        }

        function setThumbnail(data) {
          state.thumbnail = data;
        }
        return {
          props,
          state,
          setImgSource,
          show,
          onClick,
          onClickClose,
          setThumbnail
        };
      },

      template:
        '<div class="img-wall__popup" v-show="isVisible" @click="onClick($event)">\
            <div class="img-wall__popup__img" v-bind:style="styles"/>\
            <div class="img-wall__popup__close" @click="onClickClose($event)"></div>\
          </div>'
    },

    row: {
      props: ["thumbnails"],

      template:
        '<div class="img-wall__row">\
            <column v-for="thumbnail in thumbnails" :thumbnail="thumbnail"></column>\
          </div>',

      setup(props) {
        function thumbnailsByCell(cell) {
          return props.thumbnails ? props.thumbnails[cell] : [];
        }

        return {
          props,
          thumbnailsByCell
        };
      },

      components: {
        column: {
          setup(props, { emit }) {
            const state = reactive({
              styles: {
                height: "",
                width: ""
              }
            });
            function getImageUrl(name) {
              return "/assets/" + name + ".jpg";
            }

            function onClick() {
              emit(
                "zoomIn",
                getImageUrl(props.thumbnail.name),
                props.thumbnail
              ); //notify to img-wall
            }
            onMounted(() => {
              state.styles.height = "auto";
              state.styles.width = "auto";
            });
            return {
              props,
              state,
              getImageUrl,
              onClick
            };

          },

          props: ["thumbnail"],

          template:
            '<a @click="onClick" class="img-wall__thumbnail">\
                <img v-bind:src="getImageUrl(thumbnail.name)"\
                  v-bind:alt="thumbnail.name"\
                  v-bind:style="styles"/>\
              </a>'
        }
      }
    }
  },

  props: ["tournament"],

  created: function() {
    this.checkLayout();
  }
};
</script>

<style>
.img-wall {
  display: flex;
  -ms-flex-pack: center;
  justify-content: center;
  -ms-flex-direction: column-reverse;
  flex-direction: column-reverse;
  width: 100%;
  height: 100%;
  display: -ms-flexbox;
  display: flex;
  flex: 1;
  flex-basis: auto; /*UAT-6499 - IE 對預設的flex-basis:0 會有排版的問題, 因此改auto去按照內容排版*/
  overflow: hidden;
  position: absolute;
}

.img-wall__row {
}

.img-wall__thumbnail > img {
  cursor: pointer;
  max-width: 40%;
}

.img-wall__popup {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 10;
  display: -ms-flexbox;
  display: flex;
  -ms-flex-align: center;
  align-items: center;
  -ms-flex-pack: center;
  justify-content: center;
  background: #f0d47e;
  background: -webkit-linear-gradient(
    rgba(240, 212, 126, 0.7),
    rgba(206, 138, 98, 0.7)
  );
  background: -o-linear-gradient(
    rgba(240, 212, 126, 0.7),
    rgba(206, 138, 98, 0.7)
  );
  background: -moz-linear-gradient(
    rgba(240, 212, 126, 0.7),
    rgba(206, 138, 98, 0.7)
  );
  background: linear-gradient(
    rgba(240, 212, 126, 0.7),
    rgba(206, 138, 98, 0.7)
  );
}

.img-wall__popup__img {
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
  height: 100%;
  width: 100%;
  cursor: pointer;
  max-height: 120px;
}
</style>
