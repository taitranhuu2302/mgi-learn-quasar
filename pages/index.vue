<script setup lang="ts">
import {EventEmitter} from "unenv/runtime/node/events/_events";
import init = EventEmitter.init;

interface IProductResponse {
  products: IProduct[];
  total: number;
  skip: number;
  limit: number;
}

interface IProduct {
  id: number;
  title: string;
  description: string;
  price: number;
  discountPercentage: number;
  rating: number;
  stock: number;
  brand: string;
  category: string;
  thumbnail: string;
  images: string[];
}

const selected = ref([])
const columns = ref([
  {
    name: 'id',
    label: 'Id',
    align: 'left',
    field: 'id',
    format: (val: any) => `#${val}`,
    sortable: true,
  },
  {name: 'title', label: 'Title', align: 'left', field: 'title', sortable: true},
  {name: 'description', label: 'Description', align: 'left', field: 'description', sortable: true},
  {name: 'price', label: 'Price', align: 'left', field: 'price', format: (val: any) => `$${val}`, sortable: true},
  {
    name: 'discountPercentage',
    label: 'Discount',
    align: 'left',
    field: 'discountPercentage',
    format: (val: any) => `${val}%`,
    sortable: true,
  },
  {name: 'rating', label: 'Rating', align: 'left', format: (val: any) => `${val}/5`, field: 'rating', sortable: true},
  {name: 'stock', label: 'Stock', align: 'left', field: 'stock', sortable: true},
  {name: 'brand', label: 'Brand', align: 'left', field: 'brand', sortable: true},
  {name: 'category', label: 'Category', align: 'left', field: 'category', sortable: true},
])
const initialPagination = ref({
  page: 1,
  rowsPerPage: 10,
  rowsNumber: 0,
})
const skip = ref(0)
const {data, refresh, pending} = await useLazyAsyncData<IProductResponse>("GET_PRODUCT", () => {
  return $fetch<IProductResponse>(`https://dummyjson.com/products?limit=${initialPagination.value.rowsPerPage}&skip=${skip.value}`)
})
const products = ref<IProduct[]>([])
const isOpenTableSetting = ref<Boolean>(false);
const visibleColumns = ref(['id', 'title', 'description', 'price', 'discountPercentage', 'rating', 'stock', 'brand', 'category']);

if (data.value) {
  products.value = data.value.products
  initialPagination.value.rowsNumber = data.value.total
  initialPagination.value.rowsPerPage = data.value.limit
}
const getSelectedString = () => {
  return selected.value.length === 0 ? '' : `${selected.value.length} record${selected.value.length > 1 ? 's' : ''} selected of ${products.value.length}`
}

watchEffect(() => {
  if (data.value) {
    products.value = data.value.products
    initialPagination.value.rowsNumber = data.value.total
    initialPagination.value.rowsPerPage = data.value.limit
  }
})

const onRequest = (props: any) => {
  const {page, rowsPerPage, sortBy,descending} = props.pagination
  initialPagination.value.rowsPerPage = rowsPerPage
  initialPagination.value.page = page;
  skip.value = (page - 1) * rowsPerPage
  console.log(sortBy, descending)
  refresh()
}


</script>

<template>
  <div class="q-pa-md">
    <div class="row justify-end q-mb-md">
      <q-btn color="primary" @click="isOpenTableSetting = !isOpenTableSetting">Table Settings</q-btn>
      <q-dialog v-model="isOpenTableSetting">
        <q-card>
          <q-card-section class="row items-center q-pb-none">
            <div class="text-h6">Table Settings</div>
            <q-space/>
            <q-btn icon="close" flat round dense v-close-popup/>
          </q-card-section>

          <q-card-section>
            <p>View Columns</p>
            <div class="row">
              <div class="col-3">
                <q-checkbox val='id' v-model="visibleColumns" label='Id'/>
              </div>
              <div class="col-3">
                <q-checkbox val='title' v-model="visibleColumns" label='Title'/>
              </div>
              <div class="col-3">
                <q-checkbox val='description' v-model="visibleColumns" label='Description'/>
              </div>
              <div class="col-3">
                <q-checkbox val='price' v-model="visibleColumns" label='Price'/>
              </div>
              <div class="col-3">
                <q-checkbox val='discountPercentage' v-model="visibleColumns" label='Discount'/>
              </div>
              <div class="col-3">
                <q-checkbox val='rating' v-model="visibleColumns" label='Rating'/>
              </div>
              <div class="col-3">
                <q-checkbox val='stock' v-model="visibleColumns" label='Stock'/>
              </div>
              <div class="col-3">
                <q-checkbox val='brand' v-model="visibleColumns" label='Brand'/>
              </div>
              <div class="col-3">
                <q-checkbox val='category' v-model="visibleColumns" label='Category'/>
              </div>
            </div>
            <div>
              <q-checkbox v-model="visibleColumns" label='Id'/>
            </div>
          </q-card-section>
        </q-card>
      </q-dialog>
    </div>
    <q-table
      flat bordered
      title="Products"
      :rows="products"
      :columns="columns"
      :selected-rows-label="getSelectedString"
      row-key="id"
      selection="multiple"
      v-model:selected="selected"
      separator="cell"
      v-model:pagination="initialPagination"
      :visible-columns="visibleColumns"
      :loading="pending"
      @request="onRequest"
    >
      <template v-slot:header="props">
        <q-tr :props="props">
          <q-th
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
            class="text-weight-bold"
          >
            {{ col.label }}
          </q-th>
        </q-tr>
      </template>
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="id" :props="props">
            {{ props.row.id }}
          </q-td>
          <q-td key="title" :props="props">
            {{ props.row.title }}
            <q-popup-edit v-model="props.row.title" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
          <q-td key="description" :props="props">
            {{ props.row.description }}
            <q-popup-edit v-model="props.row.description" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
          <q-td key="price" :props="props">
            {{ props.row.price }}
            <q-popup-edit v-model="props.row.price" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
          <q-td key="discountPercentage" :props="props">
            {{ props.row.discountPercentage }}
            <q-popup-edit v-model="props.row.discountPercentage" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
          <q-td key="rating" :props="props">
            {{ props.row.rating }}
            <q-popup-edit v-model="props.row.rating" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
          <q-td key="stock" :props="props">
            {{ props.row.stock }}
            <q-popup-edit v-model="props.row.stock" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
          <q-td key="brand" :props="props">
            {{ props.row.brand }}
            <q-popup-edit v-model="props.row.brand" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
          <q-td key="category" :props="props">
            {{ props.row.category }}
            <q-popup-edit v-model="props.row.category" v-slot="scope">
              <q-input v-model="scope.value" dense autofocus counter @keyup.enter="scope.set"/>
            </q-popup-edit>
          </q-td>
        </q-tr>
      </template>
    </q-table>
  </div>
</template>

<style scoped lang="sass">
</style>