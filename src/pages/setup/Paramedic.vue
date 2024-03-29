<template>
  <q-page class="page-app">
    <q-card
      square
      class="icard"
    >
      <q-toolbar class="entry-caption">
        <strong>{{ pagetitle }}</strong>
        <q-space />
        <q-input
          dark
          v-model="filter"
          standout
          rounded
          dense
          outline
          debounce="500"
          label-color="white"
          placeholder="Pencarian"
        >
          <template v-slot:append>
            <q-icon
              v-if="filter === ''"
              name="search"
              size="sm"
            />
            <q-icon
              v-else
              name="clear"
              class="cursor-pointer"
              size="sm"
              @click="filter = ''"
            />
          </template>
        </q-input>
      </q-toolbar>
      <q-table
        dense
        square
        :rows="data"
        :columns="columns"
        no-data-label="data kosong"
        no-results-label="data yang kamu cari tidak ditemukan"
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
        table-class="fit-table-ui"
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
                    @click="runMethod(btn.onclick, props.row.sysid)"
                  >
                    <q-tooltip content-class="tooltips-app">
                      {{ btn.tooltips }}
                    </q-tooltip>
                  </q-icon>
                </div>
                <div v-else-if="col.name === 'is_internal'">
                  <q-toggle
                    v-model="props.row.is_internal"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_permanent'">
                  <q-toggle
                    v-model="props.row.is_permanent"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_transfer'">
                  <q-toggle
                    v-model="props.row.is_transfer"
                    true-value="1"
                    false-value="0"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'is_email_reports'">
                  <q-toggle
                    v-model="props.row.is_email_reports"
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
                <div v-else-if="col.name === 'create_date'">
                  {{ $INDDateTime(props.row.create_date) }}
                </div>
                <div v-else-if="col.name === 'update_date'">
                  {{ $INDDateTime(props.row.update_date) }}
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
        <q-space />
        <q-select
          v-model="paramedic_group"
          :options="paramedic_groups"
          option-label="label"
          option-value="value"
          emit-value
          map-options
          outlined
          label="Dokter/Paramedik"
          style="min-width: 200px"
          dense
          options-dense
          square
          class="bg-white text-white"
          @update:model-value="loaddata()"
        />
      </q-toolbar>
    </q-page-sticky>

    <!-- Dialog UI -->
    <q-dialog
      v-model="dataevent"
      persistent
      transition-show="flip-down"
      transition-hide="flip-up"
    >
      <q-card
        class="icard"
        style="width: 700px; max-width: 90vw"
        square
      >
        <q-bar class="entry-caption">
          {{ title }}
          <q-space />
          <q-btn
            v-close-popup
            dense
            glossy
            rounded
            icon="close"
            color="red-5"
            size="xs"
          >
            <q-tooltip>Close</q-tooltip>
          </q-btn>
        </q-bar>
        <q-card-section class="q-gutter-sm">
          <div class="row items-center q-col-gutter-sm q-mb-sm">
            <div class="col-3">
              <q-input
                v-model="edit.paramedic_code"
                dense
                outlined
                square
                label="Kode"
                stack-label
              />
            </div>
            <div class="col-7">
              <q-input
                v-model="edit.paramedic_name"
                dense
                outlined
                square
                label="Nama Dokter/Paramedic"
                stack-label
              />
            </div>
            <div class="col-2">
              <q-checkbox
                v-model="edit.is_active"
                true-value="1"
                false-value="0"
                dense
                label="Aktif"
                stack-label
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-xs">
            <div class="col-3">
              <q-select
                v-model="edit.paramedic_type"
                :options="paramedic_groups"
                outlined
                dense
                options-dense
                label="Kelompok Paramedik"
                option-value="value"
                option-label="label"
                emit-value
                map-options
                fill-input
                stack-label
                square
              />
            </div>
            <div class="col-7">
              <q-select
                v-model="edit.price_group"
                :options="price_groups"
                outlined
                dense
                options-dense
                label="Grup tarif jasa"
                option-value="sysid"
                option-label="group_name"
                emit-value
                map-options
                fill-input
                stack-label
                square
              />
            </div>
          </div>
        </q-card-section>
        <div style="height: 55vh">
          <q-tabs
            v-model="tab"
            dense
            no-caps
            align="justify"
            narrow-indicator
            class="bg-teal-10 text-white shadow-2"
          >
            <q-tab
              name="general"
              icon="fas fa-address-card"
              label="Umum"
            />
            <q-tab
              name="bank"
              icon="fas fa-money-bill"
              label="Akun Bank"
            />
            <q-tab
              name="configuration"
              icon="fas fa-cog"
              label="Seting"
            />
          </q-tabs>
          <q-tab-panels
            v-model="tab"
            animated
            swipeable
            vertical
            transition-prev="jump-up"
            transition-next="jump-up"
          >
            <q-tab-panel name="general">
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-4">
                  <q-input
                    v-model="edit.dob"
                    dense
                    outlined
                    square
                    label="Tanggal Lahir"
                    stack-label
                    type="date"
                  />
                </div>
              </div>
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-12">
                  <q-input
                    v-model="edit.address"
                    dense
                    outlined
                    square
                    label="Alamat"
                    stack-label
                    type="textarea"
                    input-style="max-height:40px"
                  />
                </div>
              </div>
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-6">
                  <q-input
                    v-model="edit.phone1"
                    dense
                    outlined
                    square
                    label="Telepon 1"
                    stack-label
                  />
                </div>
                <div class="col-6">
                  <q-input
                    v-model="edit.phone2"
                    dense
                    outlined
                    square
                    label="Telepon 2"
                    stack-label
                  />
                </div>
              </div>
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-6">
                  <q-input
                    v-model="edit.handphone1"
                    dense
                    outlined
                    square
                    label="Handphone 1"
                    stack-label
                  />
                </div>
                <div class="col-6">
                  <q-input
                    v-model="edit.handphone2"
                    dense
                    outlined
                    square
                    label="Handphone 2"
                    stack-label
                  />
                </div>
              </div>
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-12">
                  <q-input
                    v-model="edit.email_personal"
                    dense
                    outlined
                    square
                    label="Email"
                    stack-label
                  />
                </div>
              </div>
            </q-tab-panel>
            <q-tab-panel name="bank">
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-12">
                  <q-input
                    v-model="edit.bank_name"
                    dense
                    outlined
                    square
                    label="Nama Bank"
                    type="textarea"
                    autogrow
                    stack-label
                  />
                </div>
              </div>
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-12">
                  <q-input
                    v-model="edit.account_name"
                    dense
                    outlined
                    square
                    label="Pemilik rekening"
                    type="textarea"
                    autogrow
                    stack-label
                  />
                </div>
              </div>
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-12">
                  <q-input
                    v-model="edit.account_number"
                    dense
                    outlined
                    square
                    label="Nomor rekening"
                    type="textarea"
                    autogrow
                    stack-label
                  />
                </div>
              </div>
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-6">
                  <q-toggle
                    v-model="edit.is_transfer"
                    true-value="1"
                    false-value="0"
                    dense
                    outlined
                    square
                    label="Honor dokter/paramedik ditransfer"
                    stack-label
                  />
                </div>
              </div>
            </q-tab-panel>
            <q-tab-panel name="configuration">
              <div class="row items-start q-col-gutter-sm q-mb-sm">
                <div class="col-12">
                  <q-input
                    v-model="edit.dept_name"
                    dense
                    outlined
                    square
                    label="Klinik Rawat Jalan (Default)"
                    stack-label
                    readonly
                  >
                    <template v-slot:append>
                      <q-icon
                        name="search"
                        color="green-10"
                        size="sm"
                        @click="dlgClinic = true"
                      />
                    </template>
                  </q-input>
                </div>
              </div>
              <div class="row items-center q-col-gutter-sm q-mb-sm">
                <div class="col-12">
                  <q-select
                    v-model="edit.specialist_sysid"
                    :options="specialists"
                    outlined
                    dense
                    options-dense
                    label="Spesialisasi Dokter/Paramedik"
                    option-value="sysid"
                    option-label="specialist_name"
                    emit-value
                    map-options
                    fill-input
                    stack-label
                    square
                  />
                </div>
              </div>
              <div class="row items-center q-col-gutter-sm q-mb-sm">
                <div class="col-6">
                  <q-toggle
                    v-model="edit.is_internal"
                    true-value="1"
                    false-value="0"
                    dense
                    outlined
                    square
                    label="Dokter Dalam (Praktek)"
                    stack-label
                  />
                </div>
                <div class="col-6">
                  <q-toggle
                    v-model="edit.is_permanent"
                    true-value="1"
                    false-value="0"
                    dense
                    outlined
                    square
                    label="Dokter Tetap (Purna Waktu)"
                    stack-label
                  />
                </div>
              </div>
              <div class="row items-center q-col-gutter-sm q-mb-sm">
                <div class="col-6">
                  <q-input
                    v-model="edit.email"
                    dense
                    outlined
                    square
                    label="Alamat email (Honor Dokter/Paramedic)"
                    stack-label
                  />
                </div>
                <div class="col-6">
                  <q-toggle
                    v-model="edit.is_email_reports"
                    true-value="1"
                    false-value="0"
                    dense
                    outlined
                    square
                    label="Honor dokter diemail"
                    stack-label
                  />
                </div>
              </div>
            </q-tab-panel>
          </q-tab-panels>
        </div>
        <q-separator />
        <q-card-section
          class="dialog-action q-pa-sm"
          position="bottom"
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
    <department
      v-if="dlgClinic"
      :show="dlgClinic"
      enumtype="OUTPATIENT"
      @CloseData="getClinic"
    />
  </q-page>
</template>

<script>
import department from 'components/master/Department.vue'
import { defineComponent, ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useQuasar, QSpinnerIos } from 'quasar'

export default defineComponent({
  name: 'Paramedic',
  components: { department },
  setup() {
    const $q = useQuasar()
    const $store = useStore()
    const $router = useRouter()
    const loading = ref(false)
    const edit = ref({})
    const dataevent = ref(false)
    const title = ref('Tambah Data')
    const filter = ref('')
    const tab = ref('general')
    const url = ref('')
    const photo = ref(null)
    const ktp = ref(null)
    const sim = ref(null)
    const kartu_keluarga = ref(null)
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

    const btn_loading = ref(false)
    const pagetitle = ref('')
    const api_url = ref({})
    const btns = ref([])
    const access = ref({})

    const dlgSpecialist = ref(false)
    const dlgClinic = ref(false)
    const paramedic_group = ref('DOCTOR')
    const paramedic_groups = ref([
      { value: 'DOCTOR', label: 'Dokter' },
      { value: 'NURSE', label: 'Perawat' },
      { value: 'PHARMACIST', label: 'Apoteker' },
      { value: 'OTHERS', label: 'Lain-Lain' }
    ])

    const price_groups = ref([])
    const specialists = ref([])

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
          paramedic_type: paramedic_group.value,
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
      loading.value = false
      dataevent.value = true
      edit.value = {
        sysid: -1,
        paramedic_code: '',
        paramedic_name: '',
        paramedic_type: 'DOCTOR',
        employee_id: '',
        price_group: null,
        email: '',
        is_internal: true,
        is_permanent: true,
        is_transfer: false,
        is_email_reports: false,
        tax_number: '',
        bank_name: '',
        accout_name: '',
        account_number: '',
        cityzen_number: '',
        bpjs_number: '',
        is_active: true,
        dept_sysid: -1,
        dept_name: '',
        specialist_sysid: null,
        dob: null,
        address: '',
        phone1: '',
        phone2: '',
        handphone1: '',
        handphone2: '',
        email_personal: ''
      }
    }

    async function edit_event(primary = '') {
      if (selected.value.length > 0 || !(primary === '')) {
        if (primary === '') {
          let item = selected.value[0]
          primary = item.sysid
        }
        loading.value = false
        let props = {}
        props.url = api_url.value.edit
        props.sysid = primary
        edit.value = {}
        ktp.value = ''
        sim.value = ''
        photo.value = ''
        let respon = await $store.dispatch('master/GET_DATA', props)
        if (!(typeof respon === 'undefined')) {
          dataevent.value = true
          edit.value = respon
        }
      }
    }

    async function delete_event(primary = '') {
      if (selected.value.length > 0 || !(primary === '')) {
        if (primary === '') {
          let item = selected.value[0]
          primary = item.sysid
        }
        $q.dialog({
          title: 'Konfirmasi',
          message: 'Apakah data ini akan di hapus ?',
          cancel: true,
          persistent: true
        }).onOk(() => {
          let json = {}
          json.sysid = primary
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
      let sysid = -1
      try {
        let app = {}
        app.data = edit.value
        app.url = api_url.value.save
        loading.value = true
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
            $q.loading.hide()
            $q.notify({
              color: 'negative',
              textcolor: 'white',
              message: msg,
              position: 'top',
              timeout: 2000
            })
          }
        }
      } finally {
        loading.value = false
      }
    }

    async function loaddata() {
      selected.value = []
      onRequest({
        pagination: pagination.value,
        filter: filter.value
      })
    }

    function runMethod(method, primary = '') {
      this[method](primary)
    }

    function getClinic(closed, data) {
      dlgClinic.value = closed
      if (!(typeof data.sysid == 'undefined')) {
        edit.value.dept_sysid = data.sysid
        edit.value.dept_name = data.dept_code + ' - ' + data.dept_name
      }
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
      let props = {}
      props.url = 'setup/price-group/open'
      $store.dispatch('master/GET_DATA', props).then((response) => {
        price_groups.value = response
      })
      props.url = 'setup/specialist/open'
      $store.dispatch('master/GET_DATA', props).then((response) => {
        specialists.value = response
      })
      loaddata()
    })

    return {
      loading,
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
      tab,
      dlgClinic,
      dlgSpecialist,
      runMethod,
      onRequest,
      add_event,
      edit_event,
      delete_event,
      loaddata,
      save_data,
      btn_loading,
      kartu_keluarga,
      paramedic_group,
      paramedic_groups,
      price_groups,
      specialists,
      getClinic
    }
  }
})
</script>
