<template>
  <div class="al-c pa-3">
    <q-btn class="mr-2" flat dense @click="toggleMenu('left')">
      <img src="/img/ic-menu.svg" width="22px" />
    </q-btn>
    <span class="fz-18 mr-auto">Chat</span>

    <div class="al-c mr-1">
      <q-btn
        class="ml-3"
        flat
        dense
        v-for="it in list"
        :key="it.icon"
        @click="onAct(it)"
      >
        <img :src="`/img/${it.icon}.svg`" width="22px" />
      </q-btn>
    </div>
  </div>
</template>

<script>
export default {
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