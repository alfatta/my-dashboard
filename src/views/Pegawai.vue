<script setup>
import { NCard, NDataTable, NButton } from 'naive-ui';
import { h } from 'vue';

const data = [
  { n: 1, nama: 'Andi', usia: 24, jabatan: 1 },
  { n: 2, nama: 'Budi', usia: 23, jabatan: 3 },
  { n: 3, nama: 'Caca', usia: 22, jabatan: 2 },
  { n: 4, nama: 'Dedi', usia: 21, jabatan: 2 },
  { n: 5, nama: 'Enda', usia: 20, jabatan: 3 },
]

const onEditPegawai = (pegawai) => {
  alert('EDIT : ' + pegawai.nama)
}

const onDeletePegawai = (pegawai) => {
  alert('DELETE : ' + pegawai.nama)
}

const columns = [
  { title: "No", key: 'n' },
  { title: "Nama", key: 'nama' },
  { title: "Usia", key: 'usia' },
  { title: "Jabatan", key: 'jabatan' },
  {
    title: "Action",
    key: 'action',
    render: (row) => {
      return [
        h(
          NButton,
          {
            type: 'info',
            ghost: true,
            style: { marginRight: '10px' },
            onClick: () => onEditPegawai(row)
          },
          { default: () => 'Edit' }
        ),
        h(
          NButton,
          {
            type: 'error',
            ghost: true,
            onClick: () => onDeletePegawai(row)
          },
          { default: () => 'Delete' }
        )
      ]
    }
  },
]
</script>

<template>
  <NCard title="Pegawai">
    <h1>Ini halaman pegawai</h1>

    <NDataTable 
      :data="data"
      :columns="columns" />
  </NCard>
</template>
