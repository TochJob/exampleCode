<template>
  <section class="store">
    <EFilter
      class="store__filter"
      :isSearch="true"
      :isPeriod="true"
      :isBrand="false"
      :isStore="false"
      searchPlaceholder="Поиск бренда"
      @changeInput="changeInput"
      @selectedOption="selectedOption"
      :listForSearch="listForSearch"
    />
    <EChapter title="Бренды" :tableData="talbeData" :chartData="chartData" />
  </section>
</template>
<script>
import EChapter from "@/components/ExternalAnalytics/EChapter.vue";
import EFilter from "@/components/ExternalAnalytics/EFilter.vue";
import { CustomTable } from "@/components/UI/CustomTable";
import { mapActions, mapGetters, mapMutations } from "vuex";

export default {
  components: { EFilter, CustomTable, EChapter },
  data() {
    return {
      choosedId: "",
      choosedPlate: "",
      talbeData: [],
      chartData: [],
      listForSearch: [],
      limit: 40,
      offset: 0,
    };
  },
  methods: {
    ...mapActions([
      "GET_CURRENT_BRAND",
      "GET_BRAND_FROM_SEARCH",
      "GET_FILTRED_CHART",
      "EDIT_CHOOSED_OPTION",
    ]),
    ...mapMutations(["change_params"]),
    async getChartInfo(data) {
      console.log("getChartInfo", data);
      await this.change_params({
        brands_ids: [data.id],
        metric: data?.sort || "gmv",
      });
      const response = await this.GET_FILTRED_CHART();
      this.chartData = response;
    },
    selectedOption(item) {
      console.log(item);
      this.EDIT_CHOOSED_OPTION(item);
    },
    async firstLoad(data) {
      const storeData = await this.GET_CURRENT_BRAND(data);
      this.talbeData = storeData.items;
    },

    async changeInput(text) {
      let data = {
        from: "brand",
        text: text,
      };
      this.listForSearch = await this.GET_BRAND_FROM_SEARCH(data);
    },
  },
  computed: {
    ...mapGetters(["choosedOption", "selectedChartItem", "filtredDate"]),
  },
  watch: {
    filtredDate(item) {
      const data = {
        id: this.choosedId,
        sort: this.choosedPlate || item.value,
        limit: this.limit,
        offset: this.offset,
      };
      this.getChartInfo(data);
      this.firstLoad(data);
    },
    selectedChartItem(item) {
      const data = {
        id: this.choosedId,
        sort: item.value,
      };
      this.choosedPlate = item.value;
      this.getChartInfo(data);
    },
    choosedOption(item) {
      let data = {
        limit: this.limit,
        offset: this.offset,
        id: item.id,
        sort: this.choosedPlate,
      };
      this.firstLoad(data);
      this.choosedId = item.id;
      this.getChartInfo(data);
    },
  },
  mounted() {
    this.firstLoad({
      limit: this.limit,
      offset: this.offset,
    });
  },
};
</script>

<style lang="scss">
.store {
  .filter {
    align-items: flex-end;
    padding-bottom: 20px;
  }
}
</style>
