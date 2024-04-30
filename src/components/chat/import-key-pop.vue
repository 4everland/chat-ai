<template>
  <qs-popup v-model="showPop" hide-close title="Import API Key">
    <div class="mt-2">
      <q-input v-model="form.name" label="Key Name" outlined dense></q-input>
      <q-input
        v-model="form.value"
        label="API Key"
        class="mt-5"
        outlined
        dense
      ></q-input>
    </div>
    <div class="ta-r mt-6">
      <q-btn flat class="bd-1" @click="showPop = false">Cancel</q-btn>
      <q-btn class="ml-3" color="primary" @click="onSave">Save</q-btn>
    </div>
  </qs-popup>
</template>

<script>
import { mapState } from "vuex";

export default {
  computed: {
    ...mapState({
      importKey: (s) => s.importKey,
    }),
  },
  data() {
    return {
      showPop: false,
      form: {
        name: "",
        value: "",
      },
    };
  },
  watch: {
    showPop(val) {
      if (val && this.importKey) {
        this.form = { ...this.importKey };
      }
    },
  },
  created() {
    this.$bus.on("show-import", () => {
      this.showPop = true;
    });
  },
  methods: {
    onSave() {
      let msg = "";
      const { name, value } = this.form;
      if (!name) msg = "Key name is required";
      else if (!value) msg = "API Key is required";
      if (msg) {
        return this.$toast(msg);
      }
      this.$setStore({
        apiKey: value,
        importKey: { ...this.form },
      });
      this.showPop = false;
    },
  },
};
</script>