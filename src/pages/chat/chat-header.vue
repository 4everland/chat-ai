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
    <span class="fz-18 mr-auto line-1">{{ title || "Chat" }}</span>

    <div class="al-c mr-1">
      <choose-key-btn />
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
import { mapGetters, mapState } from "vuex";
export default {
  computed: {
    ...mapState(["isLeftOpen", "isRightOpen", "chatLogs"]),
    ...mapGetters(["chatMenu"]),
    title() {
      return this.chatMenu?.title;
    },
  },
  data() {
    return {
      list: [
        {
          icon: "ic-download",
          name: "download",
        },
        {
          icon: "ic-upload",
          name: "upload",
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
      } else if (name == "download") {
        this.onDownload();
      }
    },
    onDownload() {
      const json = JSON.stringify(
        {
          ...this.chatMenu,
          logs: this.chatLogs,
        },
        null,
        "  "
      );
      const name = "4ever-chat " + new Date().format();
      window.download(json, name + ".json", "application/json");
    },
  },
};
</script>