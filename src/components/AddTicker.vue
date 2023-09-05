<template>
  <section>
    <p>Ticker</p>
    <div class="flex items-center">
      <input v-model="ticker" @keydown.enter="add" type="text" class="input" />
      <add-button @click="add" type="button" :disabled="disabled" />
    </div>
  </section>
</template>

<script>
import AddButton from "@/components/AddButton.vue";

export default {
  name: "AddTicker",
  components: { AddButton },

  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  emits: {
    "add-ticker": (value) => typeof value === "string" && value.length,
  },

  data() {
    return { ticker: "" };
  },

  methods: {
    add() {
      if (!this.ticker.length) return;
      this.$emit("add-ticker", this.ticker);
      this.ticker = "";
    },
  },
};
</script>
