<template>
  <div class="al-c">
    <q-btn
      icon="edit"
      dense
      flat
      size="sm"
      class="mr-1"
      v-if="importKey && importKey.value == apiKey"
      @click="showImport"
    ></q-btn>
    <q-btn
      flat
      dense
      class="bg-white bd-1"
      :loading="loading"
      @click="isEpand = !isEpand"
    >
      <span class="fz-13 mr-auto px-2 ta-l lh-15" style="min-width: 120px">{{
        keyName
      }}</span>
      <img
        src="/img/ic-down.svg"
        width="14"
        class="icon-down trans-200 mr-1"
        :class="[
          {
            'up-down': isEpand,
          },
        ]"
      />
      <q-menu
        transition-show="jump-down"
        transition-hide="jump-up"
        @before-hide="isEpand = false"
      >
        <div v-if="apiKey" class="pa-3 pb-1 fz-12 gray">API Key</div>
        <q-list dense style="min-width: 160px">
          <q-item
            clickable
            v-close-popup
            v-for="it in myKeyList"
            :key="it.id"
            :active="it.key == apiKey"
            @click="setKey(it.key)"
          >
            <q-item-section>
              <span>{{ it.name }}</span>
            </q-item-section>
          </q-item>
          <q-item
            clickable
            v-close-popup
            v-if="!keyList.length"
            @click="onCreate"
          >
            <q-item-section> Create API Key </q-item-section>
          </q-item>
          <q-item clickable v-close-popup v-if="!importKey" @click="showImport">
            <q-item-section> Import API Key </q-item-section>
          </q-item>
        </q-list>
      </q-menu>
    </q-btn>
  </div>
</template>

<script>
import { mapState } from "vuex";

export default {
  computed: {
    ...mapState({
      token: (s) => s.loginData.token,
      keyList: (s) => s.keyList,
      apiKey: (s) => s.apiKey,
      importKey: (s) => s.importKey,
    }),
    myKeyList() {
      const list = [...this.keyList];
      if (this.importKey) {
        const { name, value } = this.importKey;
        list.push({
          name: name + ` - (import)`,
          key: value,
        });
      }
      return list;
    },
    keyName() {
      const item = this.myKeyList.find((it) => it.key == this.apiKey);
      if (item) return item.name;
      return "Choose Key";
    },
  },
  data() {
    return {
      isEpand: false,
      loading: false,
      hasGot: false,
    };
  },
  watch: {
    isEpand(val) {
      if (val) this.getList();
    },
    importKey(val) {
      if (!val) {
        this.hasGot = false;
        this.getList();
      }
    },
  },
  created() {
    this.getList();
    this.$bus.on("tip-key", () => {
      this.onTip();
    });
  },
  methods: {
    setKey(apiKey) {
      this.$setStore({
        apiKey,
      });
    },
    showImport() {
      this.$bus.emit("show-import");
    },
    goApiManage() {
      window.open(this.$getHomeUrl("/ai-rpc?tab=Keys"));
    },
    async onCreate() {
      try {
        this.loading = true;
        await this.$http.post("/rpc/ai/manager/keys", {
          name: "autoGen",
          limit: "",
        });
        await this.getList();
      } catch (error) {
        console.log(error);
      }
      this.loading = false;
    },
    async getList() {
      if (!this.token) return;
      try {
        const { data } = await this.$http.get("/rpc/ai/manager/keys");
        this.$setStore({
          keyList: data.items,
        });
        let apiKey = this.apiKey;
        if (apiKey) {
          const isIn = this.myKeyList.find((it) => it.key == apiKey);
          if (!isIn) apiKey = "";
        }
        if (!apiKey) {
          apiKey = this.myKeyList[0]?.key || "";
        }
        this.setKey(apiKey);
        if (!this.keyList.length && !this.hasGot) {
          // this.onTip();
          this.hasGot = true;
          this.onCreate();
        }
      } catch (error) {
        console.log(error);
      }
    },
    async onTip() {
      try {
        await this.$confirm("API Key required", {
          okLabel: "Create",
        });
        await this.onCreate();
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>