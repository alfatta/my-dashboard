<script setup>
import { NCard, NDataTable, NButton, NSpace, NModal, NForm, NFormItem, NInput } from 'naive-ui';
import { h, ref, onMounted, computed } from 'vue';
import axios from 'axios'
axios.defaults.baseURL = 'https://10c6-140-213-134-214.ngrok.io'

const data = ref([])
const currentPage = ref(1)
const isLoading = ref(false)
const totalPage = ref(1)
const perPage = 10
const filter = ref()

const editId = ref(null)
const formData = ref({
  nama: '',
  usia: 0,
  jabatan: 0
})

const showModal = ref(false)

const deleteLoadingId = ref(null)

const onEditPegawai = (pegawai) => {
  editId.value = pegawai.id
  formData.value.nama = pegawai.nama
  formData.value.usia = pegawai.usia
  formData.value.jabatan = pegawai.jabatan
  showModal.value = true
}

const resetForm = () => {
  editId.value = null
  formData.value.nama = ''
  formData.value.usia = 0
  formData.value.jabatan = 0
  showModal.value = false
}

const onSubmitForm = () => {
  if (editId.value) {
    editPegawai()
  } else {
    insertPegawai()
  }
}

const onDeletePegawai = (pegawai) => {
  if (!deleteLoadingId.value) {
    deletePegawai(pegawai.id)
  }
}

const insertPegawai = () => {
  axios({
    method: 'post',
    // baseURL: 'https://10c6-140-213-134-214.ngrok.io',
    url: '/pegawai',
    data: formData.value
  }).then((response) => {
    resetForm()
    getPegawai(1)
  }).catch((error) => {
    console.log(error.message)
  })
}
const editPegawai = () => {
  axios({
    method: 'patch',
    // baseURL: 'https://10c6-140-213-134-214.ngrok.io',
    url: '/pegawai/' + editId.value,
    data: formData.value
  }).then((response) => {
    resetForm()
    getPegawai(currentPage.value)
  }).catch((error) => {
    console.log(error.message)
  })
}
const deletePegawai = (id) => {
  deleteLoadingId.value = id
  axios({
    method: 'delete',
    // baseURL: 'https://10c6-140-213-134-214.ngrok.io',
    url: '/pegawai/' + id
  }).then((response) => {
    getPegawai(currentPage.value)
  }).catch((error) => {
    console.log(error.message)
  }).finally(() => {
    deleteLoadingId.value = null
  })
}
const getPegawai = (page) => {
  currentPage.value = page
  isLoading.value = true
  axios({
    method: 'get',
    // baseURL: 'https://10c6-140-213-134-214.ngrok.io',
    url: '/pegawai',
    params: {
      _page: page,
      _limit: perPage,
      _sort: 'id',
      _order: 'desc',
      nama: filter.value
    }
  }).then((response) => {
    totalPage.value = Math.ceil(response.headers['x-total-count'] / perPage)
    data.value = response.data.map((item, i) => {
      item.no = ((page - 1) * 10) + (i + 1)
      return item
    })
  }).catch((error) => {
    console.log(error.message)
  }).finally(() => {
    isLoading.value = false
  })
}

onMounted(() => {
  getPegawai(1)
})

const columns = [
  { title: "No", key: 'no', size: 2 },
  { title: "Nama", key: 'nama' },
  { title: "Usia", key: 'usia' },
  { title: "Jabatan", key: 'jabatan' },
  {
    title: "Action",
    key: 'action',
    render: (row) => {
      return h(NSpace, {}, [
        h(
          NButton,
          {
            type: 'info',
            ghost: true,
            onClick: () => onEditPegawai(row)
          },
          { default: () => 'Edit' }
        ),
        h(
          NButton,
          {
            type: 'error',
            ghost: true,
            onClick: () => onDeletePegawai(row),
            loading: row.id == deleteLoadingId.value
          },
          { default: () => 'Delete' }
        )
      ])
    }
  },
]

const pagination = computed(() => {
  return {
    pageSize: perPage,
    pageCount: totalPage.value,
    page: currentPage.value
  }
})

const handlePageChange = (page) => {
  getPegawai(page)
}
</script>

<template>
  <NCard title="Pegawai">
    <NButton @click="showModal = true">Add New</NButton>
    <NDataTable 
      remote
      :loading="isLoading"
      :data="data"
      :columns="columns"
      :pagination="pagination"
      @update:page="handlePageChange" />
  </NCard>
  <NModal v-model:show="showModal">
    <NCard style="width: 50%" title="Data Pegawai">
      <NForm
        @submit.prevent="onSubmitForm"
        :model="formData"
        label-placement="left"
        label-width="auto"
        require-mark-placement="right-hanging"
        size="medium">
        <NFormItem label="Nama" path="nama">
          <NInput v-model:value="formData.nama" />
        </NFormItem>
        <NFormItem label="Usia" path="usia">
          <NInput v-model:value.number="formData.usia" />
        </NFormItem>
        <NFormItem label="Jabatan" path="jabatan">
          <NInput v-model:value.number="formData.jabatan" />
        </NFormItem>
        <NFormItem label="&nbsp;">
          <NButton type="info" ghost attr-type="submits">Save</NButton>
        </NFormItem>
      </NForm>
    </NCard>
  </NModal>
</template>
