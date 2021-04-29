<template>
  <pdfexport ref="gridPdfExport">
    <Grid
      :data-items="dataResult"
      :columns="columns"
      :group="group"
      :groupable="groupable"
      :filterable="filterable"
      :filter="filter"
      :reorderable="reorderable"
      :sortable="sortable"
      :sort="sort"
      :pageable="{ buttonCount: 4, pageSizes: true }"
      :skip="skip"
      :take="take"
      :detail="detailComponent"
      @datastatechange="dataStateChange"
      @columnreorder="columnReorder"
      :expand-field="'expanded'"
      @expandchange="expandChange"
    >
      <template v-slot:myTemplate="{ props }">
        <DataItem :data-item="props.dataItem" />
      </template>
      <toolbar>
        <button
          title="Export to Excel"
          class="k-button k-primary"
          @click="exportExcel"
        >
          Export to Excel
        </button>
        <button class="k-button k-primary" @click="exportPDF">
          Export to PDF
        </button>
      </toolbar>
    </Grid>
  </pdfexport>
</template>

<script>
// @ is an alias to /src
import { Grid, GridToolbar } from "@progress/kendo-vue-grid";
import { process } from "@progress/kendo-data-query";
import { GridPdfExport } from "@progress/kendo-vue-pdf";
import { saveExcel } from "@progress/kendo-vue-excel-export";
import DataItem from "../components/DataItem.vue";

export default {
  name: "Home",
  components: {
    Grid,
    toolbar: GridToolbar,
    pdfexport: GridPdfExport,
    DataItem,
  },
  data: () => ({
    items: [],
    dataResult: [],
    skip: 0,
    take: 10,
    reorderable: true,
    filterable: true,
    filter: null,
    sort: [],
    sortable: true,
    group: [],
    groupable: true,
    dataItem: "expandedItem",
    detailComponent: "myTemplate",
    columns: [
      { field: "beneficiaries" },
      { field: "face_value", filter: "numeric" },
      { field: "issuing_bank_name" },
      { field: "reference_number" },
      { field: "facility" },
      { field: "issuance_date", filter: "date" },
      { field: "updated_at", filter: "date" },
      { field: "expiry_date", filter: "date" },
    ],
  }),
  async created() {
    const { skip, take, filter, sort } = this; // , take, sort, group
    const dataState = { skip, take, filter, sort };

    await this.axios.get("http://localhost:3000/list").then(({ data }) => {
      this.items = this.formatData(data);
      this.dataResult = process(this.items, dataState);
    });
  },
  methods: {
    createAppState(dataState) {
      this.take = dataState.take;
      this.skip = dataState.skip;
      if (dataState.group) {
        dataState.group.map((group) => (group.aggregates = this.aggregates));
      }
      this.group = dataState.group;
      this.filter = dataState.filter;
      this.sort = dataState.sort;
    },
    dataStateChange(event) {
      this.createAppState(event.data);
      const { skip, take, filter, sort, group } = this;
      this.dataResult = process(this.items, {
        skip,
        take,
        filter,
        sort,
        group,
      });
    },
    pageChangeHandler(event) {
      this.skip = event.page.skip;
      this.take = event.page.take;
    },
    columnReorder(options) {
      this.columns = options.columns;
    },
    exportExcel() {
      saveExcel({
        data: this.items,
        fileName: "myFile",
        columns: this.columns,
      });
    },
    exportPDF() {
      const tempSort = this.sort;

      this.sort = null;

      this.$nextTick(() => {
        const { skip, take, filter, sort } = this;
        this.$refs.gridPdfExport.save(
          process(this.items, { skip, take, filter, sort })
        );
        this.sort = tempSort;
      });
    },
    expandChange(event) {
      const isExpanded =
        event.dataItem.expanded === undefined
          ? event.dataItem.aggregates
          : event.dataItem.expanded;
      event.dataItem.expanded = !isExpanded;
    },
    formatData(data) {
      return data.map((item) => ({
        ...item,
      }));
    },
  },
};
</script>
