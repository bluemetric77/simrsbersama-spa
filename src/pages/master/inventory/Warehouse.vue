<template>
  <q-page class="page-app">
    <q-card
      square
      class="icard q-mb-sm"
    >
      <q-bar class="entry-caption">Tipe Gudang/Lokasi</q-bar>
      <q-card-section clas="q-pa-xs">
        <q-btn-toggle
          v-model="warehouse_group"
          style="border: 1px solid #027be3"
          no-caps
          rounded
          unelevated
          toggle-color="primary"
          color="white"
          text-color="primary"
          :options="[
            { label: 'Gudang Medis', value: 'MEDICAL' },
            { label: 'Gudang Umum', value: 'GENERAL' },
            { label: 'Gudang Gizi/Dapur', value: 'NUTRITION' }
          ]"
          @update:model-value="loaddata()"
        />
      </q-card-section>
    </q-card>

    <q-card
      square
      class="icard"
    >
      <q-bar class="entry-caption">
        <strong>{{ pagetitle }}</strong>
        <q-space />
        <q-input
          v-model="filter"
          dense
          outline
          debounce="500"
          label-color="white"
          borderless
          placeholder="Pencarian"
          input-class="text-white"
        >
          <template v-slot:append>
            <q-icon
              v-if="filter === ''"
              name="search"
              size="sm"
              color="white"
            />
            <q-icon
              v-else
              name="clear"
              class="cursor-pointer"
              size="sm"
              color="white"
              @click="filter = ''"
            />
          </template>
        </q-input>
      </q-bar>
      <q-table
        dense
        square
        :rows="data"
        :columns="columns"
        no-data-label="data kosong"
        no-results-label="data yang cari tidak ditemukan"
        row-key="sysid"
        :filter="filter"
        separator="cell"
        selection="single"
        v-model:selected="selected"
        v-model:pagination="pagination"
        binary-state-sort
        @request="onRequest"
        :loading="loading"
        virtual-scroll
        table-class="fit-table-ui-with-parameter-0"
      >
        <template v-slot:loading>
          <q-inner-loading showing>
            <q-spinner-ball
              size="75px"
              color="red-10"
            />
          </q-inner-loading>
        </template>

        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
            >
              {{ col.label }}
            </q-th>
          </q-tr>
        </template>
        <template v-slot:body="props">
          <q-tr
            :props="props"
            @click="props.selected = !props.selected"
          >
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
            >
              <div class="grid-data">
                <div v-if="col.name === 'action'">
                  <q-icon
                    v-for="(btn, index) in btns"
                    v-show="btn.is_allowed && btn.is_show"
                    :key="index"
                    no-caps
                    flat
                    class="q-mr-sm"
                    :name="btn.icon"
                    size="xs"
                    color="green-10"
                    @click="runMethod(btn.onclick, props.row.uuid_rec)"
                  >
                    <q-tooltip content-class="tooltips-app">
                      {{ btn.tooltips }}
                    </q-tooltip>
                  </q-icon>
                </div>
                <div v-else-if="col.name === 'is_received'">
                  <q-toggle
                    v-model="props.row.is_received"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_sales'">
                  <q-toggle
                    v-model="props.row.is_sales"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_distribution'">
                  <q-toggle
                    v-model="props.row.is_distribution"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_active'">
                  <q-toggle
                    v-model="props.row.is_active"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_direct_purchase'">
                  <q-toggle
                    v-model="props.row.is_direct_purchase"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_production'">
                  <q-toggle
                    v-model="props.row.is_production"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'create_date'">
                  {{ $INDDateTime2(props.row.create_date) }}
                </div>
                <div v-else-if="col.name === 'update_date'">
                  {{ $INDDateTime2(props.row.update_date) }}
                </div>
                <div v-else>
                  {{ col.value }}
                </div>
              </div>
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </q-card>

    <!-- place QPageSticky at end of page -->
    <q-page-sticky
      expand
      position="top"
    >
      <q-toolbar class="main-toolbar">
        <q-btn
          v-for="(btn, index) in btns"
          :key="index"
          dense
          no-caps
          flat
          v-show="btn.is_allowed"
          class="btn-toolbar q-mr-xs"
          :icon="btn.icon"
          :label="btn.caption"
          @click="runMethod(btn.onclick)"
        >
          <q-tooltip content-class="tooltips-app">
            {{ btn.tooltips }}
          </q-tooltip>
        </q-btn>
      </q-toolbar>
    </q-page-sticky>

    <!-- Dialog UI Interface-->
    <q-dialog
      v-model="dataevent"
      persistent
      transition-show="flip-down"
      transition-hide="flip-up"
    >
      <q-card
        class="icard"
        square
        style="width: 600px; max-width: 90vw"
      >
        <q-bar class="entry-caption">
          {{ title }}
          <q-space />
          <q-btn
            v-close-popup
            dense
            flat
            rounded
            icon="close"
            color="red-5"
            size="sm"
          >
            <q-tooltip>Tutup</q-tooltip>
          </q-btn>
        </q-bar>

        <q-card-section class="q-gutter-sm">
          <div class="row items-center q-col-gutter-sm q-mb-sm">
            <div class="col-4">
              <q-input
                v-model="edit.location_code"
                dense
                outlined
                square
                label="Kode Gudang"
                stack-label
              />
            </div>
            <div class="col-8">
              <q-input
                v-model="edit.location_name"
                dense
                outlined
                square
                label="Nama Gudang/Lokasi"
                stack-label
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.inventory_account"
                dense
                outlined
                square
                label="Akun Inventory/stock"
                stack-label
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    size="sm"
                  />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.cogs_account"
                dense
                outlined
                square
                label="Akun HPP/COGS"
                stack-label
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    size="sm"
                  />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.cost_account"
                dense
                outlined
                square
                label="Akun Biaya"
                stack-label
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    size="sm"
                  />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.variant_account"
                dense
                outlined
                square
                label="Akun Varian"
                stack-label
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    size="sm"
                  />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-select
                v-model="edit.warehouse_type"
                dense
                outlined
                square
                label="Tipe Gudang"
                :options="['Gudang Utama', 'Gudang Unit']"
                options-dense
                stack-label
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-6">
              <q-checkbox
                v-model="edit.is_sales"
                true-value="1"
                false-value="0"
                dense
                outlined
                square
                label="Diizinkan untuk penjualan"
                stack-label
              />
            </div>
            <div class="col-6">
              <q-checkbox
                v-model="edit.is_direct_purchase"
                true-value="1"
                false-value="0"
                dense
                outlined
                square
                label="Diizinkan pembelian tunai"
                stack-label
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-6">
              <q-checkbox
                v-model="edit.is_received"
                true-value="1"
                false-value="0"
                dense
                outlined
                square
                label="Diizinkan untuk penerimaan PO"
                stack-label
              />
            </div>
            <div class="col-6">
              <q-checkbox
                v-model="edit.is_production"
                true-value="1"
                false-value="0"
                dense
                outlined
                square
                label="Diizinkan produksi"
                stack-label
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-checkbox
                v-model="edit.is_distribution"
                true-value="1"
                false-value="0"
                dense
                outlined
                square
                label="Diizinkan untuk distribusi"
                stack-label
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-checkbox
                v-model="edit.is_active"
                true-value="1"
                false-value="0"
                dense
                outlined
                square
                label="Status gudang (Aktif)"
                stack-label
              />
            </div>
          </div>
        </q-card-section>
        <q-separator />
        <q-card-section
          class="dialog-action q-pa-sm"
          align="right"
        >
          <q-btn
            class="q-mr-sm"
            icon="save"
            label="Simpan"
            flat
            no-caps
            @click="save_data()"
          >
            <template v-slot:loading>
              <q-spinner class="on-left" />
              Proses
            </template>
          </q-btn>
          <q-btn
            icon="undo"
            label="Batal"
            no-caps
            flat
            v-close-popup
          />
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useQuasar } from 'quasar'

export default defineComponent({
  name: 'PriceClass',
  setup() {
    const $q = useQuasar()
    const $store = useStore()
    const $router = useRouter()

    const edit = ref({})
    const dataevent = ref(false)
    const title = ref('Tambah Data')
    const filter = ref('')
    const loading = ref(false)
    const warehouse_group = ref('MEDICAL')

    const pagination = ref({
      sortBy: 'sysid',
      descending: false,
      page: 1,
      rowsPerPage: 20,
      rowsNumber: 20
    })
    const data = ref([])
    const selected = ref([])
    const columns = ref([])

    const pagetitle = ref('')
    const api_url = ref({})
    const btns = ref([])
    const access = ref({})

    async function onRequest(props) {
      let { page, rowsPerPage, rowsNumber, sortBy, descending } =
        props.pagination
      let filter = props.filter

      let fetchCount = rowsPerPage === 0 ? rowsNumber : rowsPerPage
      loading.value = true
      try {
        let props = {
          page: page,
          limit: fetchCount,
          filter: filter,
          sortBy: sortBy,
          descending: descending,
          group_name: warehouse_group.value,
          url: api_url.value.retrieve
        }
        let respon = await $store.dispatch('master/GET_DATA', props)
        data.value = respon.data
        pagination.value = {
          rowsNumber: respon.total,
          page: respon.current_page,
          rowsPerPage: respon.per_page,
          sortBy: sortBy,
          descending: descending
        }
      } catch (error) {
      } finally {
        loading.value = false
      }
    }

    async function add_event() {
      dataevent.value = true
      title.value = 'Tambah Data'
      edit.value = {
        location_code: '',
        location_name: '',
        inventory_account: '',
        cogs_account: '',
        expense_account: '',
        variant_account: '',
        warehouse_type: 'Gudang utama',
        is_sales: '1',
        is_distribution: '0',
        is_received: '1',
        is_direct_purchase: '0',
        is_production: '0',
        warehouse_group: warehouse_group.value,
        is_active: '1',
        uuid_rec: ''
      }
    }

    async function edit_event(uuidrec = '') {
      if (selected.value.length > 0 || !(uuidrec === '')) {
        if (uuidrec === '') {
          uuidrec = selected.value[0].uuid_rec
        }
        let props = {}
        props.url = api_url.value.edit
        props.uuid_rec = uuidrec
        props.progress = true
        let respon = await $store.dispatch('master/GET_DATA', props)
        if (!(typeof respon === 'undefined')) {
          title.value = 'Ubah Data'
          dataevent.value = true
          edit.value = respon
        }
      }
    }

    async function delete_event(uuidrec = '') {
      if (selected.value.length > 0 || !(uuidrec === '')) {
        if (uuidrec === '') {
          uuidrec = selected.value[0].uuid_rec
        }
        $q.dialog({
          title: 'Konfirmasi',
          message: 'Apakah data ini akan di hapus ?',
          cancel: true,
          persistent: true
        }).onOk(() => {
          let json = {}
          json.uuid_rec = uuidrec
          json.url = api_url.value.delete
          $store.dispatch('master/DELETE_DATA', json).then((respon) => {
            if (!(typeof respon === 'undefined')) {
              let msg = respon.data
              if (respon.success) {
                dataevent.value = false
                $q.notify({
                  color: 'positive',
                  textcolor: 'white',
                  message: msg,
                  position: 'top',
                  timeout: 2000
                })
                loaddata()
              } else {
                $q.notify({
                  color: 'negative',
                  textcolor: 'white',
                  message: msg,
                  position: 'top',
                  timeout: 2000
                })
              }
            }
          })
        })
      }
    }

    async function save_data() {
      let app = {}
      app.data = edit.value
      app.operation = edit.value.sysid === -1 ? 'inserted' : 'updated'
      app.url = api_url.value.save
      app.progress = true
      let respon = await $store.dispatch('master/POST_DATA', app)
      if (!(typeof respon === 'undefined')) {
        let msg = respon.data
        if (respon.success) {
          dataevent.value = false
          $q.notify({
            color: 'positive',
            textcolor: 'white',
            message: msg,
            position: 'top',
            timeout: 2000
          })
          loaddata()
        } else {
          $q.notify({
            color: 'negative',
            textcolor: 'white',
            message: msg,
            position: 'top',
            timeout: 2000
          })
        }
      }
    }

    async function loaddata() {
      selected.value = []
      onRequest({
        pagination: pagination.value,
        filter: filter.value
      })
    }

    function runMethod(method, uuidrec = '') {
      this[method](uuidrec)
    }

    onMounted(async () => {
      let property = await $store.dispatch(
        'home/GET_PAGEPROPERTY',
        $router.currentRoute.value.fullPath
      )
      columns.value = property.columns
      pagetitle.value = property.title
      api_url.value = property.url
      btns.value = property.btn
      access.value = property.access
      loaddata()
    })

    return {
      data,
      edit,
      dataevent,
      title,
      filter,
      pagination,
      selected,
      columns,
      pagetitle,
      api_url,
      btns,
      access,
      loading,
      warehouse_group,
      runMethod,
      onRequest,
      add_event,
      edit_event,
      delete_event,
      loaddata,
      save_data
    }
  }
})
</script>
