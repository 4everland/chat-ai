<style lang="scss">
.model-item {
  &.check {
    background: #fff;
    &:hover {
      background: #e9eff5;
    }
  }
  &.uncheck {
    background: #f8fafc;
    color: #94a3b8;
    &:hover {
      // background: #F8FAFC;
    }
  }
}
</style>

<template>
  <div class="h-flex h100p">
    <div class="pa-3">
      <div class="al-c">
        <span class="fz-18 ml-2">Model List</span>

        <q-btn class="ml-auto" dense flat :loading="loadingModel">
          <q-icon name="add_circle_outline" size="22px"></q-icon>
          <q-menu
            max-height="360px"
            transition-show="jump-down"
            transition-hide="jump-up"
            :offset="[50, 0]"
          >
            <div class="pa-3 pb-0 tiny-input pos-s top-0 bg-white z-10">
              <q-input
                v-model="searchKey"
                outlined
                dense
                placeholder="Search Models"
              ></q-input>
            </div>
            <div class="pa-4 ta-c" v-if="!modelGroups.length">
              <span class="fz-14 gray">No Results</span>
            </div>
            <q-list dense separator style="min-width: 160px">
              <template v-for="group in modelGroups" :key="group.name">
                <div class="gray fz-12 px-4 py-2 mt-3">
                  {{ group.name }}
                </div>
                <q-item
                  v-for="row in group.subs"
                  :key="row.id"
                  clickable
                  :active="selected.includes(row.id)"
                  active-class="bg-f1 gray-3-"
                  @click="onSelect(row.id)"
                >
                  <q-item-section>
                    <span class="py-1">{{ row.name }}</span>
                  </q-item-section>
                </q-item>
                <q-separator />
              </template>
            </q-list>
          </q-menu>
        </q-btn>

        <q-btn class="ml-3" dense flat @click="onClose">
          <q-icon name="close" size="22px"></q-icon>
        </q-btn>
      </div>
    </div>
    <q-scroll-area
      class="flex-1"
      :thumb-style="{
        right: '2px',
        width: '3px',
        opacity: 0.35,
      }"
    >
      <div class="pa-3">
        <div
          v-for="it in modelOptions"
          :key="it.id"
          class="px-3 py-2 bdrs-6 mb-3 model-item pos-r hover-wrap"
          :class="checked.includes(it.id) ? 'check' : 'uncheck'"
        >
          <div class="al-c">
            <jazz-icon :hash="it.id" :size="30"></jazz-icon>
            <span class="ml-2">{{ it.name }}</span>
          </div>
          <div
            class="y-center right-0 mr-3 pr-10 bg-white bdrs-6 al-c hover-show"
          >
            <q-toggle v-model="checked" :val="it.id" size="sm" />
            <!-- v-show="$inDev" -->
            <img
              src="/img/settings.svg"
              width="20"
              class="hover-1 mr-2"
              @click="onConfig(it)"
            />
          </div>
        </div>
      </div>
    </q-scroll-area>
    <div class="pa-3 ta-r">
      <q-btn v-show="!!selected.length" dense flat @click="onClear">
        <img src="/img/trash.svg" width="22" />
      </q-btn>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapState } from "vuex";

export default {
  data() {
    return {
      loadingModel: false,
      selected: [],
      checked: [],
      searchKey: "",
    };
  },
  computed: {
    ...mapState({
      aiModels: (s) => s.aiModels,
    }),
    ...mapGetters(["chatMenu"]),
    modelGroups() {
      const groups = [];
      for (const row of this.aiModels) {
        if (this.searchKey) {
          const reg = new RegExp(this.searchKey, "i");
          if (!reg.test(row.name)) continue;
        }
        const { tokenizer } = row.architecture;
        let group = groups.find((it) => it.name == tokenizer);
        if (!group) {
          group = { name: tokenizer, subs: [], num: 0 };
          groups.push(group);
        }
        group.subs.push(row);
      }
      return groups;
    },
    modelOptions() {
      return this.selected
        .map((id) => {
          return this.aiModels.find((it) => it.id == id);
        })
        .filter((it) => !!it);
    },
  },
  watch: {
    checked(val) {
      // this.$emit("update-checked", val);
      this.$setState({
        checkModelIds: val,
      });
      this.$store.commit("updateChatMenu", {
        checkedModels: val,
      });
    },
    selected(val) {
      this.$store.commit("updateChatMenu", {
        selectedModels: val,
      });
    },
    "chatMenu.id"() {
      this.onInit();
    },
  },
  created() {
    let { model = "" } = this.$route.query;
    this.initModel = model;
    this.onInit();
    this.$bus.on("select-model", (id) => {
      this.onSelect(id);
    });
  },
  methods: {
    onClose() {
      this.$bus.emit("toggleMenu", "right");
    },
    async onInit() {
      const {
        selectedModels = [],
        checkedModels = [],
        modelConfig = {},
      } = this.chatMenu || {};
      this.$setState({
        configMap: modelConfig,
      });
      this.selected = [...selectedModels];
      this.checked = [...checkedModels];
      await this.getModels();
      let model = this.initModel;
      if (model) {
        this.initModel = "";
        this.$router.replace("/");
      }
      if (model) {
        const isIn = this.aiModels.find((it) => it.id == model);
        if (!isIn) model = "";
      }
      if (!this.selected.length && !model) {
        model = "openai/gpt-3.5-turbo"; // "4ever/auto"; //
      }
      if (model) {
        if (!this.selected.includes(model)) {
          this.onSelect(model);
        } else if (!this.checked.includes(model)) {
          this.checked = [...this.checked, model];
        }
      }
    },
    onConfig(it) {
      this.$setState({
        configModelId: it.id,
      });
    },
    onClear() {
      this.selected = [];
      this.checked = [];
    },
    onSelect(id) {
      const selected = [...this.selected];
      const checked = [...this.checked];
      let index = selected.indexOf(id);
      if (index == -1) {
        selected.push(id);
        checked.push(id);
      } else {
        selected.splice(index, 1);
        index = checked.indexOf(id);
        if (index > -1) checked.splice(index, 1);
      }
      this.selected = selected;
      this.checked = checked;
    },
    async getModels() {
      if (this.aiModels.length) {
        if (Date.now() - localStorage.t_ai_models < 3 * 3600e3)
          return console.log("models", this.aiModels.length);
      }
      try {
        console.log("get models");
        this.loadingModel = true;
        const { data } = await this.$axios.get(
          "https://ai.api.4everland.org/api/v1/models"
        );
        const rows = data.data.map((it) => {
          // if (it.id == "openrouter/auto") it.id = "4ever/auto";
          delete it.description;
          return it;
        });
        this.$setStore({
          aiModels: rows,
        });
        localStorage.t_ai_models = Date.now();
      } catch (error) {
        console.log(error);
      }
      this.loadingModel = false;
    },
  },
};
</script>