<template>
  <div class="chapter">
    <h1 class="chapter__title">{{ title }}</h1>
    <transition-group name="fadeModal">
      <template v-if="rowDefs?.length === 1">
        <div :key="choosedOption.id" class="chapter__chips">
          <div class="chapter__chips-item">{{ choosedOption.category }}</div>
        </div>
        <div :key="2" class="box chapter__box">
          <ESwitcherPlate
            class="hiddenInfo__switcher"
            v-model="activeChart"
            @getActiveFilter="getActiveChart"
            :data="chartTypes"
          />
          <BarChart class="hiddenInfo__chart" :height="300" :data="changeChartInfo" />
        </div>
      </template>
    </transition-group>
    <CustomTable
      :gridApi.sync="gridApi"
      :columnApi.sync="columnApi"
      :columnDefs="columnDefs"
      :rowHeight="70"
      :rowData="rowDefs"
      :pagination="false"
      :domLayout="true"
      :isLoading="isTableLoading"
    />
  </div>
</template>
<script>
  import { mapActions, mapGetters, mapMutations } from "vuex";
  import { CustomTable } from "../UI/CustomTable";
  import CategoryCell from "@/components/TableComponents/CategoryCell.vue";
  import EChartBox from "./EChartBox.vue";
  import ESwitcherPlate from "./ESwitcherPlate.vue";
  import BarChart from "@/components/UI/CChart.vue";
  import moment from "moment";

  export default {
    name: "EChapter",
    props: {
      title: {
        type: String,
      },
      tableData: {
        type: Object,
      },
      chartData: {
        type: Object,
      },
    },
    components: { CustomTable, CategoryCell, EChartBox, ESwitcherPlate, BarChart },
    data() {
      return {
        isTableLoading: false,
        gridApi: null,
        columnApi: null,
        currentTableInfo: [],
        activeChart: "",
        chartTypes: [
          {
            title: "Выручка",
            value: "gmv",
            isActive: true,
          },
          {
            title: "Продажи",
            value: "sales",
            isActive: false,
          },
          {
            title: "Средняя цена",
            value: "wSoldPrice",
            isActive: false,
          },
          {
            title: "Продукты",
            value: "productsAvailable",
            isActive: false,
          },
          {
            title: "С продажами",
            value: "soldProductsShare",
            isActive: false,
          },
          {
            title: "Средние продажи",
            value: "specificSales",
            isActive: false,
          },
          {
            title: "Магазины",
            value: "merchantsTotal",
            isActive: false,
          },
          {
            title: "Отзывы",
            value: "productsReviews",
            isActive: false,
          },
        ],
      };
    },
    methods: {
      ...mapActions(["GET_CURRENT_BRAND"]),
      ...mapMutations(["change_selected_chart"]),
      firstLoad() {
        this.change_selected_chart(this.chartTypes[0]);
      },
      getActiveChart(item) {
        this.currentChart = item;
        this.change_selected_chart(item);
      },
    },
    watch: {
      tableData() {
        this.currentTableInfo = this.tableData;
      },
    },

    computed: {
      ...mapGetters(["selectedChartItem", "choosedOption"]),
      columnDefs() {
        return [
          {
            field: "category",
            headerName: "Бренд",
            width: 229,
            cellRenderer: "CategoryCell",
          },
          {
            field: "profit",
            headerName: "Выручка",
            width: 230,
          },
          {
            field: "purchases",
            headerName: "Продажи",
            width: 230,
          },
          {
            field: "avg_price",
            headerName: "Ср. Цена",
            width: 230,
          },
          {
            field: "products",
            headerName: "Продукты",
            width: 230,
          },
          {
            field: "with_sales",
            headerName: "С продажами",
            width: 230,
          },
          {
            field: "average_sales",
            headerName: "СР. Продажи",
            width: 230,
          },
        ];
      },
      rowDefs() {
        let rowData = [];
        this.currentTableInfo?.forEach((item) => {
          const row = {
            category: item?.category?.name || item?.merchant?.name || item?.brand?.name,
            id: item?.category?.id || item?.merchant?.id || item?.brand?.id,
            url: item?.category?.url || item?.merchant?.url || item?.brand?.url,
            profit: `${item.gmv.toLocaleString("ru")} ₽`,
            purchases: item.sales.toLocaleString("ru"),
            avg_price: `${item.wSoldPrice.toLocaleString("ru")} ₽`,
            products: item.productsAvailable.toLocaleString("ru"),
            with_sales: item.soldProductsShare.toLocaleString("ru"),
            average_sales: Math.ceil(item.specificSales).toLocaleString("ru"),
            canHasChild: item.category ? true : false,
          };
          rowData.push(row);
        });
        return rowData;
      },
      changeChartInfo() {
        let changedChart = {
          data: [],
          labels: [],
        };
        this.chartData?.forEach((item) => {
          changedChart.data.push(item[this.selectedChartItem.value]);
          changedChart.labels.push(moment(item.date).format("DD-MM-YYYY"));
        });
        return changedChart;
      },
    },
    mounted() {
      this.firstLoad();
    },
  };
</script>
<style lang="scss">
  .chapter {
    &__title {
      margin: 10px 0 25px;
    }
    &__chips {
      display: flex;
      align-items: flex-start;
      margin-bottom: 15px;
      &-item {
        font-weight: 600;
        font-size: 14px;
        cursor: pointer;
        border-radius: 3px;
        line-height: 13px;
        color: #3586ff;
        background: rgba(53, 134, 255, 0.05);
        border: 1px solid rgba(53, 134, 255, 0.4);
        padding: 9.5px 12px;
        transition: all 0.3s;
        display: flex;
        align-items: center;
        gap: 6px;
        &:hover {
          background: rgba(53, 134, 255, 0.2);
        }
      }
      &-close {
        background: rgba(53, 134, 255, 0.1);
        border-radius: 10px;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 6px;
        border-radius: 50%;
        width: 20px;
        height: 20px;
      }
    }
    .ag-theme-test {
      height: auto;
    }

    .ag-theme-test .ag-cell {
      justify-content: flex-start;
      border-right: none;
    }
    &__box {
      align-items: flex-start;
    }
  }
</style>
