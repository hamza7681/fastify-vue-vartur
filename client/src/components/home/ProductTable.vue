<template>
  <SpinnerLoader v-if="loading" />
  <div>
    <div class="p-6">
      <router-link
        class="bg-[#5EA2EA] p-2 text-white rounded"
        to="/add-product"
      >
        Add Product
      </router-link>
    </div>
    <DataTable
      v-model:filters="filters"
      :value="products"
      :paginator="true"
      :rowsPerPageOptions="[5, 10, 20, 50]"
      :rows="5"
      :globalFilterFields="['id', 'name', 'image', 'created_at']"
      :filters="true"
    >
      <template #header>
        <div class="flex justify-start">
          <span class="p-input-icon-left pl-5">
            <input
              type="text"
              placeholder="Filter"
              class="w-96 border border-[#a3d0e4] rounded-md pl-4 py-3 focus:outline-none focus:border-sky-500 focus:ring-1 focus:ring-sky-500"
              
            />
          </span>
        </div>
      </template>
      <template>
        <Column header="ID" style="width: 140px">
          <template #body="rowData">
            <p v-tooltip="rowData.data.id">
              {{ shortenAddress(rowData.data.id) }}
            </p>
          </template>
        </Column>
        <Column
          v-for="col of columns"
          :key="col.field"
          :field="col.field"
          :header="col.header"
        ></Column>

        <Column header="Image">
          <template #body="rowData">
            <div class="h-[70px] overflow-hidden">
              <img
                :src="rowData.data.picture"
                alt="hello"
                class="w-[100px] rounded-[3px]"
              />
            </div>
          </template>
        </Column>
        <Column header="Action">
          <template #body="rowData">
            <div class="flex flex-row gap-2">
              <div class="hover:bg-[#3489e5] p-2 rounded-[2px] bg-[#5EA2EA]">
                <RouterLink
                  :to="'/edit-product/' + rowData.data.id"
                  class="text-white rounded"
                >
                  <i class="pi pi-pencil" />
                </RouterLink>
              </div>
              <div class="hover:bg-[#dc3545] p-2 rounded-[2px] bg-[#f86c6b]">
                <button
                  v-on:click="deleteProduct(rowData.data.id)"
                  class="text-white rounded"
                >
                  <i class="pi pi-trash" />
                </button>
              </div>
            </div>
          </template>
        </Column>
      </template>
    </DataTable>
  </div>
</template>

<script>
import DataTable from "primevue/datatable";
import Column from "primevue/column";
import { http } from "../../axios/config";
import SpinnerLoader from "../spinner/SpinnerLoader.vue";

export default {
  data() {
    return {
      name: "",
      products: [],
      columns: [
        { field: "name", header: "Name" },
        { field: "createdAt", header: "Created at" },
      ],
      rows: 5, // number of rows per page
      imageUrl: null,
      loading: false,
    };
  },
  components: {
    DataTable,
    Column,
    SpinnerLoader,
  },
  methods: {
    async loadData() {
      this.loading = true;
      try {
        let result = await http.get("/product/get-products");
        this.products = result.data;
        this.imageUrl;
        this.loading = false;
      } catch (error) {
        console.log(error.response.data.msg);
        this.loading = false;
      }
    },
    onBtnClick(id) {
      console.log(id);
    },
    async deleteProduct(id) {
      this.loading = true;
      try {
        let result = await http.delete("/product/delete-product/" + id);
        if (result.status == 200) {
          this.$toast.success(result.data.msg);
          this.loadData();
          this.loading = false;
        }
      } catch (error) {
        console.log(error.response.data.msg);
        this.$toast.error(error.response.data.msg);
        this.loading = false;
      }
    },
    shortenAddress(id) {
      return `${id.slice(0, 5)}...${id.slice(id.length - 4)}`;
    },
  },
  mounted() {
    this.loadData();
  },
};
</script>
