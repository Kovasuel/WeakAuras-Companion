<template>
  <div id="sync" v-bind:class="{ top: aurasShown > 0 }">
    <v-button
      v-bind:class="{ spin: fetching }"
      v-if="usable"
      @click="refresh"
      type="refresh"
    >
      <i class="material-icons sync">sync</i>
      <span>{{ $t("app.refreshbutton.label" /* Fetch Updates */) }}</span>
    </v-button>
    <v-button v-else @click="gotoconfig" type="issue">
      <i class="material-icons error">error_outline</i>
      <span>{{ $t("app.refreshbutton.finishsetup" /* Finish Setup */) }}</span>
    </v-button>
    <div id="lastupdate" v-if="lastUpdate">
      {{ $t("app.refreshbutton.lastupdate" /* last update: */) }}
      <b>{{ lastUpdate | fromNow($i18n.locale) }}</b>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import Button from "./Button.vue";

export default {
  name: "refreshButton",
  props: ["usable", "fetching", "lastUpdate", "aurasShown"],
  data() {
    return {
      lastUpdateTimer: null
    };
  },
  components: { "v-button": Button },
  methods: {
    refresh() {
      this.$parent.compareSVwithWago();
    },
    gotoconfig() {
      this.$parent.configStep = 1;
    },
    scheduleTimer() {
      if (this.lastUpdateTimer) clearInterval(this.lastUpdateTimer);
      this.lastUpdateTimer = setInterval(() => {
        this.$forceUpdate();
      }, 1000 * 60);
    }
  },
  filters: {
    fromNow: (value, locale) => {
      if (!value) return "n/a";
      return moment(value)
        .locale(locale)
        .fromNow();
    }
  },
  mount() {
    this.scheduleTimer();
  },
  watch: {
    fetching() {
      this.scheduleTimer();
    }
  },
  beforeDestroy() {
    clearInterval(this.lastUpdateTimer);
  }
};
</script>

<style scoped lang="scss">
#sync {
  text-align: center;
  width: 100%;
  margin: auto;
  transition: all 0.4s ease-in-out;
}
#sync.top {
  position: relative;
  top: 10px;
}
#lastupdate {
  margin-top: 10px;
  font-size: small;
  color: #e6e6e6;
}
.btn-issue span,
.btn-refresh span {
  position: relative;
  bottom: 8px;
  line-height: 50px;
  cursor: pointer;
}
.material-icons {
  font-size: 34px;
  vertical-align: top;
  cursor: pointer;
}
.btn-refresh.spin {
  background: #ababab;
  border-color: transparent;
  color: #313131;
}
/* Spin Animation */
.spin .sync {
  animation-name: spin;
  animation-duration: 800ms;
  animation-iteration-count: infinite;
  animation-timing-function: ease-in-out;
  animation-fill-mode: forwards;
}
.btn-issue,
.btn-refresh {
  padding: 12px 15px;
  padding-left: 13px;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(-360deg);
  }
}
</style>
