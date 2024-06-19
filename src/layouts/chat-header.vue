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

    <div class="al-c mr-1 shrink-0" v-show="path == '/'">
      <div class="d-n">
        <input
          ref="file"
          type="file"
          accept="application/json"
          @input="onUploadFile"
        />
      </div>
      <!-- <choose-key-btn /> -->
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
        <b class="ml-1">{{ it.txt }}</b>
      </q-btn>
    </div>
  </div>
</template>

<script>
import md5 from "md5";
import { mapGetters, mapState } from "vuex";
export default {
  computed: {
    ...mapState([
      "isLeftOpen",
      "isRightOpen",
      "chatLogs",
      "chatMenus",
      "checkModelIds",
    ]),
    ...mapGetters(["chatMenu"]),
    path() {
      return this.$route.path;
    },
    modelLen() {
      return this.checkModelIds.length;
    },
    title() {
      if (this.path == "/") return this.chatMenu?.title;
      if (this.path == "/settings") return "Settings";
      return "";
    },
    list() {
      let txt = "";
      if (this.modelLen) {
        txt = "×" + this.modelLen;
      }
      return [
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
          txt,
        },
      ];
    },
  },
  data() {
    return {};
  },
  methods: {
    toggleMenu(side) {
      this.$bus.emit("toggleMenu", side);
    },
    onAct({ name }) {
      if (name == "model") {
        this.toggleMenu("right");
      } else if (name == "download") {
        this.doDownload();
      } else if (name == "upload") {
        this.$refs.file.click();
      }
    },
    doDownload() {
      const json = JSON.stringify(
        {
          ...this.chatMenu,
          time: Date.now(),
          logs: this.chatLogs,
        },
        null,
        "  "
      );
      const name = "4ever-chat " + new Date().format("date");
      window.download(json, name + ".json", "application/json");
    },
    onUploadFile(e) {
      const file = e.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (e) => {
        const content = e.target.result; // 读取到的文件内容
        this.setConfig(content);
      };
      reader.readAsText(file);
    },
    setConfig(content) {
      try {
        const { logs, ...menu } = JSON.parse(content);
        if (!Array.isArray(logs) || !menu.id) {
          throw new Error("Unsupported config");
        }
        menu.id = menu.id + "-" + md5(Date.now()).substring(0, 4);
        this.$setStore({
          chatMenus: [menu, ...this.chatMenus],
          menuIdx: 0,
        });
        setTimeout(() => {
          this.$setStore({
            chatLogs: logs,
          });
        }, 100);
      } catch (error) {
        window.$toast(error.message);
      }
    },
  },
};
</script>