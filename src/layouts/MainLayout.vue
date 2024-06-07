<style lang="scss">
</style>

<script setup>
import MainLogin from "./main-login.vue";
</script>

<template>
  <div class="">
    <main-login v-if="!token && !apiKey" />
    <q-layout v-else view="lHh Lpr lFf" class="bg-white vh100" container>
      <q-drawer
        class="bg-left"
        :width="280"
        v-model="showLeft"
        show-if-above
        :breakpoint="900"
      >
        <div>test</div>
      </q-drawer>
      <q-page-container>
        <div class="d-flex h100p">
          <div class="h100p flex-1">
            <router-view />
          </div>
        </div>
      </q-page-container>

      <q-drawer
        side="right"
        :width="280"
        v-model="showRight"
        show-if-above
        :breakpoint="900"
        style="background: #f2f5f9"
      >
        <model-list v-show="!configModelId" />
        <model-settings v-show="!!configModelId" />
      </q-drawer>
    </q-layout>
  </div>

  <import-key-pop />
</template>

<script>
import { mapState } from "vuex";
import { useQuasar } from "quasar";

export default {
  data() {
    const { screen } = useQuasar();
    const isOpen = screen.width > 900;
    return {
      showLeft: isOpen,
      showRight: isOpen,
    };
  },
  computed: {
    ...mapState({
      token: (s) => s.loginData.token,
      apiKey: (s) => s.apiKey,
      configModelId: (s) => s.configModelId,
    }),
  },
  watch: {
    showLeft(val) {
      this.$setState({
        isLeftOpen: val,
      });
    },
    showRight(val) {
      this.$setState({
        isRightOpen: val,
      });
    },
  },
  mounted() {
    this.$bus.on("toggleMenu", (side) => {
      this.onToggle(side);
    });
    this.$setState({
      isLeftOpen: this.showLeft,
      isRightOpen: this.showRight,
    });
  },
  methods: {
    onToggle(side) {
      if (side == "right") {
        this.showRight = !this.showRight;
      } else {
        this.showLeft = !this.showLeft;
      }
    },
  },
};
</script>
