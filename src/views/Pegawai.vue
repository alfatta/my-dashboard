<script setup>
import { NCard, NDataTable, NButton, NSpace } from 'naive-ui';
import { h, ref, onMounted, computed } from 'vue';
import axios from 'axios'

const data = ref([])
const currentPage = ref(1)
const isLoading = ref(false)
const totalPage = ref(1)
const perPage = 10

const deleteLoadingId = ref(null)

const onEditPegawai = (pegawai) => {
  alert('EDIT : ' + pegawai.nama)
}

const onDeletePegawai = (pegawai) => {
  if (!deleteLoadingId.value) {
    deletePegawai(pegawai.id)
  }
}

const deletePegawai = (id) => {
  deleteLoadingId.value = id
  axios({
    method: 'delete',
    baseURL: 'https://10c6-140-213-134-214.ngrok.io',
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
    baseURL: 'https://10c6-140-213-134-214.ngrok.io',
    url: '/pegawai',
    params: {
      _page: page,
      _limit: perPage
    }
  }).then((response) => {
    totalPage.value = Math.ceil(response.headers['x-total-count'] / perPage)
    data.value = response.data
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
  { title: "No", key: 'id', size: 2 },
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
    <NDataTable 
      remote
      :loading="isLoading"
      :data="data"
      :columns="columns"
      :pagination="pagination"
      @update:page="handlePageChange" />
  </NCard>
</template>
