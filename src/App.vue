<template>
  <div id="app">
    <div class="container-fluid">
      <div class="card shadow mb-4">
        <div
          class="card-header py-3 d-flex justify-content-between align-items-center"
        >
          <h6 class="m-0 font-weight-bold text-primary">Daily Stocks</h6>
          <h6 class="d-sm-none">
            <!-- <router-link to="/admin" class="btn btn-primary btn-sm d-sm-none"
              ><i class="fas fa-undo-alt mr-1"></i>Dashboard</router-link
            > -->
          </h6>
        </div>
        <div class="card-body">
          <form action="">
            <div class="form-group row">
              <div class="col-sm-6 mb-3 mb-sm-0 form-floating">
                <b-form-datepicker
                  v-model="date"
                  :min="min"
                  :max="max"
                  locale="en"
                  :hide-header="true"
                  placeholder="Date"
                  selected-variant="success"
                  today-variant="info"
                  nav-button-variant="primary"
                  ref="dateBox"
                  :class="errorDate ? 'is-invalid' : ''"
                  :date-format-options="{
                    year: 'numeric',
                    month: 'short',
                    day: 'numeric',
                    weekday: 'short',
                  }"
                  :disabled="isDateDisabled"
                ></b-form-datepicker>
                <label for="entryDate">Date</label>
              </div>
              <div class="col-sm-6 form-floating">
                <multiselect
                  v-model="category"
                  :options="catOptions"
                  :searchable="true"
                  :close-on-select="true"
                  :show-labels="true"
                  placeholder="Pick a Category"
                  track-by="name"
                  label="name"
                  @input="filterItem"
                >
                  <span slot="noResult"
                    >No Category found. Consider changing the search
                    query.</span
                  >
                </multiselect>
                <label class="multiselect__label" for="entryCategory"
                  >Pick a Category</label
                >
              </div>
            </div>
            <div class="form-group row mt-4 mb-0" v-if="rows">
              <div class="col-sm-4 mb-3 form-floating">
                <div class="pagination-container mt-sm-1">
                  <b-form-select
                    v-model="perPage"
                    :options="loadRows"
                    id="perPage"
                  ></b-form-select>
                  <label for="perPage">Set Per Page</label>
                </div>
              </div>
              <div class="col-sm-4 mb-3 form-floating">
                <div class="pagination-container">
                  <b-pagination
                    v-model="currentPage"
                    :total-rows="rows"
                    :per-page="perPage"
                    aria-controls="daily-stock"
                  ></b-pagination>
                  <label for="entryPagination" class="page-label"
                    >Pagination</label
                  >
                </div>
              </div>
              <div class="col-sm-4 mb-3 mb-sm-0 form-floating">
                <div class="pagination-container">
                  <p class="page-text">
                    {{ pageText }}
                  </p>
                  <label for="entryPageTexr">Listing Items</label>
                </div>
              </div>
            </div>
            <div class="form-group row my-0">
              <div class="col-sm-12">
                <p @click="showItemName = ''" id="entry" class="item-name">
                  {{ showItemName }}
                  <label :class="showItemName ? 'active' : ''" for="itemName"
                    >Item Name</label
                  >
                </p>
              </div>
            </div>
          </form>

          <!-- Datatable -->
          <b-datagrid
            :busy="loading"
            sticky-header
            bordered
            striped
            hover
            responsive
            id="daily-stock"
            class="editable-table"
            v-model="items"
            :fields="fields"
            :tbody-tr-class="rowClass"
            :per-page="perPage"
            :current-page="currentPage"
            @input-change="handleInput"
            @row-clicked="onRowClicked"
            :sort-by.sync="sortBy"
            :sort-desc.sync="sortDesc"
            :sort-direction="sortDirection"
          >
            <template #table-busy>
              <div class="text-center text-primary my-5">
                <!-- <b-spinner class="align-middle"></b-spinner> -->
                <div class="spinner-border align-middle"></div>
                <span class="ml-2">Loading...</span>
              </div>
            </template>
            <template #cell(name)="data">
              <span
                style="font-weight: 900"
                v-b-tooltip="{
                  container: '#daily-stock',
                }"
                :title="data.value"
                >{{ data.value }}</span
              >
            </template>
          </b-datagrid>

          <div class="row">
            <div class="col mb-3 mb-sm-0">
              <form class="" @submit.prevent=""></form>
            </div>
            <div class="col"></div>
            <div class="col">
              <!-- <router-link
                class="btn btn-primary btn-block"
                to="/admin/stock/summary"
                ><i class="fas fa-tasks mr-2"></i>Summary</router-link
              > -->
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import BEditableTable from "bootstrap-vue-editable-table";
import Multiselect from "vue-multiselect";
import itemsData from "@/itemsData.json";
import "@/assets/css/multiselect.css";

export default {
  name: "App",
  components: {
    "b-datagrid": BEditableTable,
    Multiselect,
  },
  data() {
    return {
      category: [],
      catOptions: [],
      items: [],
      rowId: "",
      sortBy: "sort_order",
      sortDesc: false,
      sortDirection: "asc",
      errorDate: false,
      min: "",
      max: "",
      isDateDisabled: false,
      date: "",
      showItemName: "",
      loading: false,
      isLoading: false,
      perPage: 25,
      currentPage: 1,
      loadRows: [
        { value: 25, text: "25" },
        { value: 50, text: "50" },
        { value: 100, text: "100" },
        { value: 200, text: "200" },
        { value: 300, text: "300" },
        { value: 400, text: "400" },
      ],
      fields: [
        // {
        //     id: 0,
        //     key: "id",
        //     label: "Slno",
        //     type: "text",
        //     editable: false,
        // },
        {
          id: 1,
          key: "name",
          label: "Item Name",
          type: "text",
          editable: false,
          class: "name-col",
          stickyColumn: false,
          variant: "info",
          sortable: true,
        },
        {
          id: 2,
          key: "size",
          label: "Size",
          type: "number",
          editable: false,
          stickyColumn: true,
          class: "size-col",
        },
        {
          id: 3,
          key: "receipt",
          label: "Receipts",
          type: "text",
          editable: true,
          class: "receipt-col",
          onfocus: "this.select()",
          validate: this.validateInput,
          inputmode: "numeric",
          pattern: "[0-9]*",
          placeholder: "Receipts",
        },
        {
          id: 4,
          key: "cb",
          label: "CB",
          type: "text",
          editable: true,
          class: "cb-col",
          onfocus: "this.value=''",
          validate: this.validateInput,
          inputmode: "numeric",
          pattern: "[0-9]*",
          placeholder: "CB",
        },
        // {
        //     id: 5,
        //     key: "wholesale_sales",
        //     label: "Wholesale Sales",
        //     type: "text",
        //     editable: true,
        //     class: "wss-col",
        //     // onfocus: "this.select()",
        //     onfocus: "this.value=''",
        //     validate: this.validateInput,
        //     inputmode: "numeric",
        //     pattern: "[0-9]*",
        //     placeholder: "Wholesale Sales",
        // },
        {
          id: 6,
          key: "ob",
          label: "OB",
          type: "number",
          editable: false,
          class: "ob-col",
        },

        {
          id: 7,
          key: "total_ob",
          label: "Total OB",
          type: "number",
          editable: false,
          class: "tob-col",
        },
        {
          id: 8,
          key: "normal_sales",
          label: "Normal Sales",
          type: "number",
          editable: false,
          class: "ns-col",
        },
        {
          id: 9,
          key: "category_name",
          label: "Category Name",
          type: "text",
          editable: false,
          class: "cat-col",
          sortable: true,
        },
      ],
      rowClass: "pending",
    };
  },
  computed: {
    rows() {
      return this.items.length;
    },
    pageText() {
      if (this.items.length > 0) {
        return `From ${Math.ceil(
          this.currentPage * this.perPage - this.perPage + 1
        )} to
                ${
                  this.items.length > this.perPage
                    ? this.perPage * this.currentPage
                    : this.items.length
                } of ${this.items.length} items (${Math.ceil(
          this.items.length / this.perPage
        )}
                Pages)`;
      } else {
        return "No Data!";
      }
    },
  },
  created() {
    this.getItems();
  },
  methods: {
    validateInput(value, data) {
      if (this.date === "") {
        this.errorDate = true;
        return {
          valid: false,
          errorMessage: "Please choose date first!",
        };
      }
      if (value === "") {
        return {
          valid: false,
          errorMessage: "Field should not be empty!",
        };
      }
      if (isNaN(value)) {
        return {
          valid: false,
          errorMessage: "Input numbers only!",
        };
      }

      const inputData = {
        ...this.items[data.index],
        [data.field.key]: data.value,
      };
      this.rowId = inputData.id;
      console.log("rowId", inputData.id);

      return {
        valid: true,
      };
    },
    handleInput(data) {
      const updatedRow = {
        ...this.items[data.index],
        [data.field.key]: data.value,
      };
      console.log("updatedRow ", updatedRow);
      console.log("updatedRow ID", updatedRow.id);
    },
    getItems() {
      this.items = itemsData.data.items;
      this.getCategory(itemsData.data.category);
    },
    getCategory(cat) {
      this.catOptions = [];
      cat.forEach((ele) => {
        this.catOptions.push({
          category_id: ele.category_id,
          name: ele.category_name,
        });
      });
    },
    async filterItem() {
      /**
       * @var this.category
       *
       */
      try {
        this.loading = true;
        let allItems = await itemsData.data.items;

        if (this.category) {
          this.items = allItems;
          let cat_id = this.category.category_id;
          const mapItem = this.items.filter((x) => x.category_id === cat_id);
          console.log("filteredItem", mapItem);
          setTimeout(() => {
            this.loading = false;
          }, 300);
          return (this.items = mapItem);
        } else {
          setTimeout(() => {
            this.loading = false;
          }, 300);
          return (this.items = allItems);
        }
      } catch (error) {
        console.log("error: ", error);
      }
      setTimeout(() => {
        this.loading = false;
      }, 500);
    },
    onRowClicked(item) {
      console.log("selectedRow: ", item.name);
      this.showItemName = item.name;
    },
  },
};
</script>

<style lang="scss" scoped>
@import "@/assets/sass/floating-label";
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

label.multiselect__label {
  position: absolute !important;
  top: -18px;
  font-size: 10px;
  color: #4d72de;
}
.form-floating > .form-control:focus,
.form-floating > .form-control:not(:placeholder-shown),
.form-floating > .form-control-plaintext:focus,
.form-floating > .form-control-plaintext:not(:placeholder-shown) {
  font-weight: normal;
}
</style>
<style lang="css">
.pagination-container {
  overflow-x: auto;
  display: flex;
  justify-content: center;
}
.pagination-container label {
  position: absolute;
  top: -16px;
  left: 30px;
  font-size: 0.669rem;
  color: #4e73df;
  font-weight: 400;
  margin: 0;
  padding: 0;
  display: block;
}
.page-text {
  width: 100%;
  height: 38px;
  display: block;
  line-height: 38px;
  border: 1px solid #d1d3e2;
  border-radius: 0.35rem;
  font-size: 1rem;
  background-clip: padding-box;
  padding: 0 9px;
  overflow-x: auto;
  white-space: nowrap;
}
@media screen and (min-width: 576px) and (max-width: 1140px) {
  .pagination-container .pagination {
    margin-right: 1rem;
    margin-left: 5rem;
  }
}
.item-name {
  position: relative;
  display: block;
  width: 100%;
  height: 43px;
  line-height: 53px;
  margin: 0 auto;
  padding: 0 10px;
  background-color: #f8f9fc;
  color: #313131;
  font-weight: 600;
  border-radius: 4px 4px 0 0;
  border-bottom: 1px solid #e3e6f0;
  box-shadow: 0 0 5px inset rgba(78, 115, 223, 0.48);
  transition: all 0.2s ease-in-out;
}
.item-name label {
  position: absolute;
  top: 0;
  left: 10px;
  color: #6f717f;
  margin: 0;
  padding: 0;
  font-size: 14px;
  transition: all 0.2s ease-in-out;
}
.item-name label.active {
  top: -18px;
  color: #4e73df;
  font-size: 0.669rem;
  font-weight: 400;
  transition: all 0.2s ease-in-out;
}
table.editable-table,
.editable-table .table {
  margin: auto;
  width: 100%;
  color: #555;
}

table.editable-table td {
  vertical-align: middle;
}

.editable-table .data-cell {
  padding: 8px;
  vertical-align: middle;
}
.editable-table thead th div {
  font-weight: bold !important;
  color: #313131;
}
.editable-table tr:hover > td {
  color: #111;
}
.editable-table tr:focus-within {
  color: #111;
  font-weight: bold;
}
/* #receipt-details .form-control {
    height: 100%;
} */

/* Set height to table */
.editable-table.b-table-sticky-header {
  overflow-y: auto;
  max-height: 493px !important;
}
.name-col {
  min-width: 120px;
  white-space: nowrap;
  overflow-x: auto;
  max-width: 130px;
  position: relative;
  scrollbar-color: #4d72de rgb(190, 188, 188);
  scrollbar-width: thin;
  background: #c8e3ea;
}
.name-col .data-cell {
  overflow-x: auto;
}
@media screen and (max-width: 575px) {
  .pagination-container label.page-label {
    top: -10px !important;
  }
  .name-col .data-cell {
    font-size: 11px;
  }
}
@media screen and (min-width: 576px) {
  .item-name {
    display: none;
  }
}
.editable-table thead th.table-info.name-col {
  background: hsl(191, 43%, 76%);
}
/* Tooltip for Name Column  */
.editable-table .tooltip.b-tooltip.bs-tooltip-bottom {
  top: -1.6rem !important;
}
.editable-table .tooltip.bs-tooltip-bottom > .arrow {
  top: 87% !important;
}
.editable-table .tooltip.b-tooltip {
  opacity: 0.7 !important;
}
.editable-table .bs-tooltip-bottom .arrow::before {
  top: -1px;
  border-width: 0.4rem 0.4rem 0;
  border-top-color: #000;
}

.editable-table thead th.table-info.name-col {
  background: hsl(191, 43%, 76%);
}

@media screen and (min-width: 1056px) {
  .name-col {
    min-width: 250px;
  }
}
.table.b-table > tbody > .table-active > td.name-col {
  background-color: #9ec9d2;
}
.receipt-col {
  width: 87px;
  background-color: rgb(255, 213, 106);
}
.receipt-col .data-cell,
.cb-col .data-cell {
  background-color: rgb(255, 213, 106);
  justify-content: end;
}

/* Receipt Details cols */
.slno-col {
  width: 70px;
}
.inv-no-col {
  width: 102px;
}
.inv-date-col {
  min-width: 130px;
}
.inv-date-col > div {
  position: relative;
}
.inv-amt-col {
  width: 113px;
}
.tds-col,
.tax-col {
  width: 96px;
}
.received-col {
  width: 130px;
}
.remarks-col {
  min-width: 150px;
}
@media screen and (min-width: 993px) {
  td.inv-date-col {
    width: 191px;
    max-width: 191px;
  }
  td.remarks-col {
    min-width: 200px;
    max-width: 200px;
  }
}
.receipt-col .form-control,
.wss-col .form-control,
.cb-col .form-control,
.rec-col {
  text-align: right;
}
.wss-col {
  width: 150px;
  background-color: rgb(168, 227, 85);
}
.wss-col .data-cell {
  background-color: rgb(168, 227, 85);
  justify-content: end;
}
.editable-table thead th.table-b-table-default.receipt-col,
.editable-table thead th.table-b-table-default.cb-col {
  background-color: rgb(255, 182, 43);
  box-shadow: inset 0 -1px 0 #e6e2e2;
}
.editable-table td.recept-col {
  background-color: rgb(255, 213, 106);
}
.editable-table thead th.table-b-table-default.wss-col {
  background-color: rgb(141, 184, 81);
  box-shadow: inset 0 -1px 0 #e6e2e2;
}
.cat-col,
.receipt-col,
.tob-col,
.ns-col,
.wss-col,
.rec-col,
.slno-col,
.inv-no-col,
.inv-date-col,
.inv-amt-col,
.tds-col,
.tax-col,
.received-col,
.remarks-col {
  white-space: nowrap;
}
.cb-col {
  width: 70px;
  min-width: 70px;
  position: relative;
}
.cb-col .data-cell {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
#receipt-details .data-cell,
#receipt-details .b-form-btn-label-control.form-control {
  min-height: 38px !important;
  max-height: 38px;
}
#receipt-details .b-form-btn-label-control.form-control > .form-control {
  margin-top: -5px;
  margin-left: -7px;
}
.editable-table thead th.cb-col > div {
  text-align: center;
}
#daily-stock table.table-editable .form-control,
.form-control {
  /* border-radius: 0 !important; */
  padding: 0.375rem 0.45rem !important;
}
.size-col .data-cell,
.ob-col .data-cell,
.tob-col .data-cell,
.ns-col .data-cell,
.wss-col .data-cell,
.rec-col .data-cell {
  justify-content: end;
}
/* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type="number"] {
  -moz-appearance: textfield;
  appearance: textfield;
}
table.dataTable tbody td {
  padding: unset;
}
.dataTable thead > tr > th.sorting,
table.dataTable thead > tr > th.sorting_asc {
  border-left: unset !important;
}
/* width */
::-webkit-scrollbar {
  width: 3px;
}
::-webkit-scrollbar:horizontal {
  height: 3px;
  position: absolute;
  left: 0;
  bottom: 4px;
  z-index: 10;
}

/* Track */
::-webkit-scrollbar-track {
  box-shadow: inset 0 0 5px grey;
  border-radius: 10px;
}

/* Handle */
::-webkit-scrollbar-thumb {
  background: #4d72de;
  border-radius: 10px;
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  background: #3757b8;
}

#receipt-detail .modal {
  padding: 1.8rem !important;
}
#receipt-detail .modal-dialog {
  max-width: 100%;
  height: 100%;
  margin: 0;
  padding: 1rem;
}
#receipt-detail .modal-content {
  border: 0;
  min-height: 100%;
  height: auto;
}
#receipt-details {
  margin-bottom: 300px;
  padding: 0;
}
#receipt-details .b-form-btn-label-control.form-control > .btn {
  display: none;
}

/* #receipt-details .table-responsive {
    padding-bottom: 100px;
}
#receipt-details .b-form-btn-label-control.form-control > .dropdown-menu {
    transform: translate3d(0px -26px 0px) !important;
} */
#receipt-details .inv-date-col > button.btn-secondary {
  color: #fff;
  background-color: #5457df;
  border-color: rgb(61, 64, 89);
}
.date-col {
  width: 200px;
  position: relative;
}
.action-col,
.action-col > .data-cell {
  justify-content: center;
}
.remove-icon {
  color: red;
  cursor: pointer;
  font-size: 20px;
  text-align: center;
}
#receipt-details .b-form-datepicker.btn-group {
  width: 100% !important;
}

.table-bordered th.br-0,
.table-bordered td.br-0 {
  border-right: 1px solid #fff !important;
}
</style>
