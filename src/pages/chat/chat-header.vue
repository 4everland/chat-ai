<template>
  <div class="al-c pa-3">
    <q-btn
      class="mr-2"
      :class="{
        'bg-btn-on': isLeftOpen,
      }"
      flat
      dense
      @click="toggleMenu('left')"
    >
      <img src="/img/ic-menu.svg" width="22" />
    </q-btn>
    <span class="fz-18 mr-auto">{{ title }}</span>

    <div class="al-c mr-1">
      <q-btn
        class="ml-3"
        :class="{
          'bg-btn-on': isRightOpen && it.name == 'model',
        }"
        flat
        dense
        v-for="it in list"
        :key="it.icon"
        @click="onAct(it)"
      >
        <img :src="`/img/${it.icon}.svg`" width="22" />
      </q-btn>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  computed: {
    ...mapState(["isLeftOpen", "isRightOpen", "chatMenus", "menuIdx"]),
    title() {
      return this.chatMenus[this.menuIdx]?.title ?? "Chat";
    },
  },
  data() {
    return {
      list: [
        {
          icon: "ic-upload",
        },
        {
          icon: "ic-download",
        },
        {
          icon: "ic-robot",
          name: "model",
        },
      ],
    };
  },
  methods: {
    toggleMenu(side) {
      this.$bus.emit("toggleMenu", side);
    },
    onAct({ name }) {
      if (name == "model") {
        this.toggleMenu("right");
      }
    },
  },
};
</script>