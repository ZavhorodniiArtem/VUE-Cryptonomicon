<template>
  <div class="wrapper">
    <div v-if="this.dataLoading">
      <p>Loading...</p>
    </div>
    <div v-else>
      <add-ticker @add-ticker="add" :disabled="tooManyTickersAdded" />
      <div class="flex">
        <div v-for="t in helpsTickers" :key="t">
          <span class="p-1 mr-2 bg-gray-200 rounded-md" @click="add(t.symbol)">
            {{ t.FullName }}
          </span>
        </div>
      </div>
      <div class="share w-[300px]" />
      <p v-if="this.errorLine" class="text-rose-600">
        This ticker is already exist
      </p>
      <div class="flex justify-between">
        <div>Filter: <input v-model="filter" /></div>
        <div class="flex">
          <button
            v-if="page > 1"
            @click="page = page - 1"
            class="bg-gray-300 px-3 py-1 rounded-md ml-3"
          >
            Назад
          </button>
          <button
            v-if="hasNextPage"
            @click="page = +page + 1"
            class="bg-gray-300 px-3 py-1 rounded-md ml-3"
          >
            Вперед
          </button>
        </div>
      </div>
      <div v-if="tickers.length" class="share" />
      <div class="cards">
        <div
          v-for="t in paginatedTickers"
          :key="t.name"
          @click="
            selectedTicker = t;
            graph = [];
          "
        >
          <div class="card" :class="{ 'active-card': selectedTicker === t }">
            <p>{{ t.name }} - USD</p>
            <p>{{ formatPrice(t.price) }}</p>
            <p @click.stop="handleDelete(t)" class="delete">Delete</p>
          </div>
        </div>
      </div>
      <div v-if="tickers.length" class="share" />
      <section v-if="selectedTicker">
        <p @click="closeGraph">Close graph</p>
        <div class="graph" ref="graph">
          <div
            v-for="(bar, idx) in normalizedGraph"
            class="bar"
            :style="{ height: `${bar || 1}%` }"
            :key="idx"
          />
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import { subscribeToTicker, unsubscribeFromTicker } from "@/api";
import AddTicker from "@/components/AddTicker.vue";

export default {
  name: "App",

  components: {
    AddTicker,
  },

  data() {
    return {
      filter: "",
      tickers: [],
      selectedTicker: null,
      graph: [],
      maxGraphElements: 1,
      page: 1,
      helpsTickers: [],
      errorLine: false,
      dataTickers: [],
      dataLoading: true,
    };
  },

  created: async function () {
    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );
    const VALID_KEYS = ["filter", "page"];
    VALID_KEYS.forEach((key) => {
      if (windowData[key]) {
        this[key] = windowData[key];
      }
    });
    const tickersData = localStorage.getItem("crypto-list");

    if (tickersData) {
      this.tickers = JSON.parse(tickersData);
      this.tickers.forEach((ticker) => {
        subscribeToTicker(ticker.name, (newPrice) =>
          this.updateTicker(ticker.name, newPrice)
        );
      });
    }
    setInterval(this.updateTicker, 5000);
    const res = await fetch(
      "https://min-api.cryptocompare.com/data/all/coinlist?summary=true"
    );
    const result = await res.json();
    this.dataTickers = result.Data;
    this.dataLoading = false;
  },

  mounted() {
    window.addEventListener("resize", this.calculateMaxGraphElements);
  },

  beforeMount() {
    window.removeEventListener("resize", this.calculateMaxGraphElements);
  },

  computed: {
    tooManyTickersAdded() {
      return this.tickers.length > 4;
    },
    startIndex() {
      return (this.page - 1) * 6;
    },
    endIndex() {
      return this.page * 6;
    },
    filteredTickers() {
      return this.tickers.filter((t) => t.name.includes(this.filter));
    },
    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
    },
    hasNextPage() {
      return this.filteredTickers.length > this.endIndex;
    },
    normalizedGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);
      if (maxValue === minValue) {
        return this.graph.map(() => 50);
      }
      return this.graph.map(
        (price) => ((price - minValue) * 100) / (maxValue - minValue)
      );
    },
    pageStateOptions() {
      return {
        filter: this.filter,
        page: this.page,
      };
    },
  },
  methods: {
    calculateMaxGraphElements() {
      if (!this.$refs.graph) {
        return;
      }
      this.maxGraphElements = this.$refs.graph.clientWidth / 24;
    },
    updateTicker(tickerName, price) {
      this.tickers
        .filter((t) => t.name === tickerName)
        .forEach((ticker) => {
          if (ticker === this.selectedTicker) {
            this.graph.push(price);
            if (this.graph.length > this.maxGraphElements) {
              this.graph.shift();
            }
          }
          ticker.price = price;
        });
    },
    formatPrice(price) {
      if (price === "-") {
        return price;
      }
      return price > 1 ? price.toFixed(2) : price.toPrecision(2);
    },
    add(ticker) {
      const currentTicker = { name: ticker, price: "-" };
      const isExisted = this.tickers.some((el) => el.name === this.ticker);
      if (isExisted) {
        this.errorLine = true;
        return;
      }
      this.tickers = [...this.tickers, currentTicker];
      this.filter = "";
      subscribeToTicker(currentTicker.name, (newPrice) =>
        this.updateTicker(currentTicker.name, newPrice)
      );
    },
    select(ticker) {
      this.selectedTicker = ticker;
    },
    /*  handleInput(event) {
      this.errorLine = false;
      const regex = new RegExp(`\\b${event.target.value}\\b`, "i");
      return Object.entries(this.dataTickers).filter((item) => {
        if (this.helpsTickers.length > 3) return;
        if (item[1].Symbol.match(regex)) {
          return this.helpsTickers.push(item[1]);
        }
      });
    },*/
    handleDelete(tickerToRemove) {
      this.tickers = this.tickers.filter((t) => t !== tickerToRemove);
      if (this.selectedTicker === tickerToRemove) {
        this.selectedTicker = null;
      }
      unsubscribeFromTicker(tickerToRemove.name);
    },
    closeGraph() {
      this.selectedTicker = null;
      this.graph = [];
    },
  },
  watch: {
    selectedTicker() {
      this.graph = [];
      this.$nextTick().then(this.calculateMaxGraphElements);
    },
    tickers() {
      localStorage.setItem("crypto-list", JSON.stringify(this.tickers));
    },
    paginatedTickers() {
      if (!this.paginatedTickers.length && this.page > 1) {
        this.page -= 1;
      }
    },
    filter() {
      this.page = 1;
    },
    pageStateOptions(value) {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}?filter=${value.filter}&page=${value.page}`
      );
    },
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.wrapper {
  display: flex;
  flex-direction: column;
  padding: 24px;
}

.input {
  width: 250px;
  margin: 8px 0;
  outline: none;
  padding: 4px 8px;
  border: 2px solid lightseagreen;
  &[type="text"] {
    border-radius: 8px;
  }
}

.share {
  display: block;
  height: 1px;
  background: lightseagreen;
  margin: 16px 0;
}

.cards {
  display: flex;
  gap: 16px;
}

.card {
  display: flex;
  flex-direction: column;
  border: 2px solid transparent;
  padding: 24px;
  border-radius: 16px;
  gap: 8px;
  cursor: pointer;
}

.card:hover {
  border: 2px solid lightseagreen;
}

.active-card {
  border: 2px solid lightseagreen;
  background: lightseagreen;
  color: white;
}

.graph {
  display: flex;
  align-items: end;
  padding: 8px;
  gap: 8px;
  height: 250px;
  border-left: 1px solid lightseagreen;
  border-bottom: 1px solid lightseagreen;
}

.bar {
  display: block;
  width: 24px;
  background: darkblue;
}
</style>
