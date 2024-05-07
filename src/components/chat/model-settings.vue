<template>
  <div class="h-flex h100p" v-if="curItem">
    <div class="pa-3 bdb-1 bg-pink0">
      <div class="al-c">
        <jazz-icon hash="test" :size="30"></jazz-icon>
        <span class="fz-15 ml-2">{{ curItem.name }}</span>

        <div class="ml-auto pr-2- bg-white- bdrs-6 al-c">
          <!-- <q-toggle v-model="checked" size="xs" /> -->
          <img
            src="/img/settings2.svg"
            width="20"
            class="hover-1 ml-1"
            @click="onBack"
          />
        </div>
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
      <div class="pa-4">
        <div class="mb-4" v-for="it in keyList" :key="it.name">
          <div v-if="!it.max">
            <div class="label-1">{{ it.label }}</div>
            <q-input
              v-model="curForm[it.name]"
              outlined
              bg-color="white"
              dense
              autogrow
              maxlength="1500"
            ></q-input>
          </div>

          <div v-else class="al-c">
            <div class="flex-1">
              <div class="al-c label-1">
                <span class="mr-1">{{ it.label }}</span>
                <q-btn icon="info_outline" flat size="xs" dense>
                  <q-tooltip
                    max-width="300px"
                    class="bg-white gray bd-1 fz-14"
                    anchor="center right"
                    self="top left"
                  >
                    <span class="fz-1">{{ it.tip }}</span>
                  </q-tooltip>
                </q-btn>
              </div>
              <q-slider
                v-model="curForm[it.name]"
                :min="it.min"
                :max="it.max"
                :step="it.step || 1"
                :name="it.name"
                dense
                track-size="3px"
                thumb-size="12px"
                thumb-color1="white"
                @update:model-value="onSlide(it, $event)"
              />
            </div>
            <q-input
              v-model="inpForm[it.name]"
              class="ml-3 tiny-input ta-r"
              bg-color="white"
              outlined
              dense
              style="width: 80px"
              @update:model-value="onInput(it, $event)"
              @blur="onBlur(it)"
            />
          </div>
        </div>

        <div class="al-c mt-4">
          <q-btn flat class="bg-white bd-1 flex-1" @click="applyToAll"
            >Apply to All</q-btn
          >
          <q-btn
            flat
            class="bg-white bd-1 flex-1 ml-3"
            :disable="!isChange"
            @click="onReset"
          >
            <span class="fz-13">Reset to Default</span>
          </q-btn>
        </div>
      </div>
    </q-scroll-area>
  </div>
</template>

<script>
import { mapState } from "vuex";

export default {
  computed: {
    ...mapState({
      aiModels: (s) => s.aiModels,
      configModelId: (s) => s.configModelId,
    }),
    curItem() {
      return this.aiModels.find((it) => it.id == this.configModelId);
    },
    isChange() {
      let change = false;
      for (const key in this.initForm) {
        if (this.initForm[key] != this.curForm[key]) {
          change = true;
          break;
        }
      }
      return change;
    },
  },
  data() {
    return {
      checked: false,
      standard: 10,
      val: "10",
      keyList: [
        {
          label: "Description (system propmt)",
          def: "",
          maxlen: 1000,
        },
        {
          tip: `This sets the upper limit for the number of tokens the model can generate in response. It won't produce more than this limit. The maximum value is the context length minus the prompt length.`,
          label: "Max Tokens",
          min: 0,
          max: 128000,
          def: 0,
          name: "maxToken",
          step: 100,
        },
        {
          tip: `Specifies the maximum number of chat message to be included in each request.`,
          label: "Chat Memory",
          min: 2,
          max: 12,
          name: "chatMemory",
          def: 4,
        },
        {
          tip: `This setting influences the variety in the model's responses. Lower values lead to more predictable and typical responses, while higher values encourage more diverse and less common responses. At 0, the model always gives the same response for a given input.`,
          label: "Temperature",
          min: 0,
          max: 2,
          name: "temperature",
          def: 1.0,
          step: 0.01,
        },
      ],
      initForm: {},
      curForm: {},
      inpForm: {},
    };
  },
  created() {
    for (const it of this.keyList) {
      this.initForm[it.name] = it.def;
    }
    this.onReset();
  },
  methods: {
    applyToAll() {
      this.$setStore({
        configMap: {
          all: {
            ...this.curForm,
          },
        },
      });
    },
    onReset() {
      this.curForm = { ...this.initForm };
      this.inpForm = { ...this.initForm };
    },
    onBlur(it) {
      if (this.inpForm[it.name] === "") {
        this.inpForm[it.name] = it.def;
      }
    },
    onInput(it, val) {
      const { name, min, max, def } = it;
      let fval = val;
      if (val === "") {
        return;
      }
      if (isNaN(val)) {
        fval = "";
      } else if (val > max) {
        fval = max;
      } else if (val < min) {
        fval = min;
      }
      if (fval === val) {
        this.curForm[name] = val;
      } else {
        this.inpForm[name] = fval;
        this.curForm[name] = val;
      }
      // console.log(name, val, fval);
    },
    onSlide(it, e) {
      this.inpForm[it.name] = e;
    },
    onBack() {
      this.$setState({
        configModelId: null,
      });
    },
  },
};
</script>