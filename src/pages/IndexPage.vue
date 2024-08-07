<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <q-form class="q-mb-xl" @submit.prevent.stop="addData">
        <q-input
          v-model="tempData.name"
          label="姓名"
          lazy-rules
          :rules="nameRules"
          required
        />
        <q-input
          v-model.number="tempData.age"
          label="年齡"
          type="number"
          lazy-rules
          :rules="ageRules"
          required
        />
        <q-btn type="submit" color="primary" class="q-mt-md">新增</q-btn>
      </q-form>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>

      <q-dialog v-model="editModalVisible" persistent>
          <q-card style="min-width: 350px">
            <q-card-section>
              <div class="text-h6">更新資料</div>
            </q-card-section>
            <q-form class="q-mb-xl" @submit.prevent.stop="patchData">
              <q-card-section class="q-pt-none">
                <q-input
                  v-model="editData.name"
                  label="姓名"
                  lazy-rules
                  :rules="nameRules"
                  required
                />
                <q-input
                  v-model.number="editData.age"
                  label="年齡"
                  type="number"
                  lazy-rules
                  :rules="ageRules"
                  required
                />
              </q-card-section>
              <q-card-actions align="right" class="text-primary">
                <q-btn flat label="取消" v-close-popup />
                <q-btn type="submit" flat label="更新" v-close-popup />
              </q-card-actions>
            </q-form>
          </q-card>
        
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios'
import { useQuasar, QTableProps } from 'quasar'
import { ref, onMounted } from 'vue'

interface btnType {
  label: string;
  icon: string;
  status: string;
}

const $q = useQuasar()
const blockData = ref([])
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
])

const nameRules = [
  val => (val && val.length > 0) || '請輸入姓名'
]

const ageRules = [
  val => !isNaN(val) || '請輸入年齡'
]

const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
])

const tempData = ref({
  name: '',
  age: '',
})

const editData = ref()
const editModalVisible = ref(false)

function handleClickOption(btn, data) {
  switch (btn.status) {
    case 'edit':
      editData.value = JSON.parse(JSON.stringify(data))
      editModalVisible.value = true
      break
    case 'delete':
      deleteData(data)
      break
  }
}

async function getData () {
  try {
    const res = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    blockData.value = res.data
  } catch {
    $q.notify({
      message: '取得資料錯誤',
    })
  }
}

async function addData () {
  try {
    const res = await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)

    if (res.data) {
      getData()
      $q.notify({
        message: '新增成功',
      })
    } else {
      $q.notify({
        message: '新增失敗',
      })
    }
  } catch (err) {
    $q.notify({
      message: '新增失敗',
    })
  }
}

async function patchData () {
  try {
    const res = await axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', editData.value)

    if (res.data) {
      getData()
      $q.notify({
        message: '更新成功',
      })
    } else {
      $q.notify({
        message: '更新失敗',
      })
    }
  } catch (err) {
    $q.notify({
      message: '更新失敗',
    })
  }
}

function deleteData (data) {
  $q.dialog({
    title: '提示',
    message: '是否確定刪除該筆資料？',
    cancel: true,
    persistent: true
  }).onOk(async () => {
    try {
      const res = await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`)

      if (res.data) {
        getData()
        $q.notify({
          message: '移除成功',
        })
      } else {
        $q.notify({
          message: '移除失敗',
        })
      }
    } catch (err) {
      $q.notify({
        message: '移除失敗',
      })
    }
  })
}

onMounted(() => {
  getData()
})
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
