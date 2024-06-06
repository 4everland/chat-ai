<style lang="scss">
</style>

<script setup>
import MainHeader from "./main-header.vue";
import MainLogin from "./main-login.vue";
</script>

<template>
  <div class="">
    <main-login v-if="!token && !apiKey" />
    <q-layout v-else view="lHh Lpr lFf" class="bg-white vh100" container>
      <q-drawer
        class="bg-left"
        :width="280"
        v-model="isOpen"
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
        v-model="isRightOpen"
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

export default {
  data() {
    return {
      isOpen: false,
      isRightOpen: false,
    };
  },
  computed: {
    ...mapState({
      token: (s) => s.loginData.token,
      apiKey: (s) => s.apiKey,
      configModelId: (s) => s.configModelId,
    }),
  },
  mounted() {
    this.$bus.on("toggleMenu", () => {
      this.onToggle();
    });
  },
  methods: {
    onToggle() {
      this.isOpen = !this.isOpen;
    },
    onClose() {
      if (this.$q.screen.xs) {
        this.isOpen = false;
      }
    },
  },
};
</script>
