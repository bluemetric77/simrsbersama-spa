<template>
  <div>
    <q-dialog
      v-model="dlgItems"
      persistent
      transition-show="scale"
      transition-hide="scale"
    >
      <q-card
        class="icard"
        style="width: 800px; max-width: 80vw; max-height: 700px"
      >
        <q-bar class="entry-caption">
          <span>Master Inventory</span>
          <q-space />
          <q-input
            v-model="filter"
            dense
            outline
            label-color="white"
            borderless
            placeholder="Pencarian"
            input-class="text-white"
            debounce="200"
          >
            <template v-slot:append>
              <q-icon name="search" color="white" size="xs" />
            </template>
          </q-input>
        </q-bar>
        <q-table
          square
          :rows="data"
          :columns="columns"
          no-data-label="data kosong"
          no-results-label="data yang kamu cari tidak ditemukan"
          row-key="item_code"
          :filter="filter"
          separator="cell"
          selection="single"
          virtual-scroll
          dense
          class="fit"
          table-style="min-height:302px; max-height: 350px"
          v-model:selected="selected"
          v-model:pagination="pagination"
          binary-state-sort
          @request="onRequest"
          :loading="loading"
        >
          <template v-slot:loading>
            <q-inner-loading showing>
              <q-spinner-ios size="70px" color="primary" />
            </q-inner-loading>
          </template>
          <template v-slot:no-data="{ icon, message, filter }">
            <div class="full-width row flex-center text-accent q-gutter-sm">
              <q-icon size="2em" name="sentiment_dissatisfied" />
              <span>{{ message }}</span>
              <q-icon size="2em" :name="filter ? 'filter_b_and_w' : icon" />
            </div>
          </template>
          <template v-slot:header="props">
            <q-tr :props="props">
              <q-th
                v-for="col in props.cols"
                :key="col.name"
                :props="props"
                class="bg-blue-grey-14 text-white"
              >
                {{ col.label }}
              </q-th>
            </q-tr>
          </template>
          <template v-slot:body="props">
            <q-tr :props="props" @click="props.selected = !props.selected">
              <q-td v-for="col in props.cols" :key="col.name" :props="props">
                <div class="grid-data">
                  <div v-if="col.name === 'item_code'">
                    <q-btn
                      :label="props.row.item_code"
                      no-caps
                      dense
                      flat
                      @click="selectdata(props.row.item_code)"
                    />
                  </div>
                  <div v-else>
                    {{ col.value }}
                  </div>
                </div>
              </q-td>
            </q-tr>
          </template>
        </q-table>
        <q-separator/>
        <q-card-section class="entry-caption q-pa-sm" align="right">
          <q-btn
            no-caps
            label="Pilih"
            color="positive"
            icon="check"
            flat
            class="q-mr-sm"
            @click="selectdata()"
          />
          <q-btn
            no-caps
            color="negative"
            label="Tutup"
            icon="close"
            flat
            @click="closedata({})"
          />
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { defineComponent, ref, onMounted } from "vue";
import { useStore } from "vuex";

export default defineComponent({
  name: "items",
  props: { show: Boolean, state: String },
  setup(props, context) {
    const $store = useStore();
    const dlgItems = ref(false);
    const pagination = ref({
      sortBy: "item_code",
      descending: false,
      page: 1,
      rowsPerPage: 25,
      rowsNumber: 25,
    });

    const filter = ref("");
    const selected = ref([]);
    const columns = ref([
      {
        name: "item_code",
        align: "left",
        label: "Kode",
        field: "item_code",
        sortable: true,
      },
      {
        name: "descriptions",
        align: "left",
        label: "Item Inventory",
        field: "descriptions",
        sortable: true,
      },
      {
        name: "item_group_name",
        align: "left",
        label: "Jenis",
        field: "item_group_name",
        sortable: true,
      },
    ]);

    const vlastcolumn = ref("");
    const data = ref([]);
    const loading = ref(false);

    async function loaddata() {
      await onRequest({
        pagination: pagination.value,
        filter: filter.value,
      });
    }
    function selectdata(account='') {
      if ((selected.value.length > 0) || (account!=='')){
        let row=null
        if (account!==''){
           data.value.forEach(el=>{
            if (account===el.item_code) {
              row =el
            }
           }) 
        } else {
          let item = selected.value[0];
          row = item;
        }
        closedata(row);
      }
    }

    async function onRequest(props) {
      let { page, rowsPerPage, rowsNumber, sortBy, descending } =
        props.pagination;
      let filter = props.filter;

      let fetchCount = rowsPerPage === 0 ? rowsNumber : rowsPerPage;
      loading.value = true;
      try {
        let prop = {
          page: page,
          limit: fetchCount,
          filter: filter,
          descending: descending,
          sortBy: sortBy,
          url: "master/inventory/items/open",
        };
        let respon = await $store.dispatch("master/GET_DATA", prop);
        data.value = respon.data;
        pagination.value = {
          rowsNumber: respon.total,
          page: respon.current_page,
          sortBy: sortBy,
          descending: descending,
          rowsPerPage: respon.per_page,
        };
      } catch (error) {
      } finally {
        loading.value = false;
      }
    }

    function closedata(record) {
      dlgItems.value = false;
      filter.value = "";
      data.value = [];
      context.emit("CloseItems", false, record);
    }

    onMounted(async () => {
      dlgItems.value = props.show;
      loaddata();
    });

    return {
      loading,
      data,
      vlastcolumn,
      filter,
      selected,
      columns,
      pagination,
      selectdata,
      onRequest,
      closedata,
      dlgItems,
    };
  },
});
</script>
