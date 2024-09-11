<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => !!val || '姓名不得為空']"
        />
        <q-input
          v-model="tempData.age"
          label="年齡"
          type="number"
          :rules="[
            (val) => !!val || '年齡不得為空',
            (val) =>
              (val && val > 0 && Number.isInteger(Number(val))) ||
              '年齡必須是正整數',
          ]"
        />
        <q-btn color="primary" class="q-mt-md" @click="handleAddOrEdit">
          {{ tempData.id !== null ? '更新' : '新增' }}
        </q-btn>
      </div>

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
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { QTableProps, useQuasar } from 'quasar';
import axios from 'axios';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

// 定義API返回的數據項的接口
interface DataItem {
  id: number;
  name: string;
  age: number;
  [key: string]: any; // 為了允許其他可能的屬性
}

// 定義包含isEditing的數據項接口
interface DataItemWithEditing extends DataItem {
  isEditing: boolean;
}

const $q = useQuasar();
const nameError = ref('');
const ageError = ref('');
const blockData = ref<DataItemWithEditing[]>([]);
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
]);

const tableButtons = ref<btnType[]>([
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
]);

const tempData = ref({
  id: null as number | null,
  name: '',
  age: '',
});

const apiUrl = 'https://dahua.metcfire.com.tw/api/CRUDTest';

onMounted(async () => {
  await fetchData();
});

async function fetchData() {
  try {
    const response = await axios.get<DataItem[]>(`${apiUrl}/a`);
    blockData.value = response.data.map((item: DataItem) => ({
      ...item,
      isEditing: false,
    }));
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

async function handleAddOrEdit() {
  if (tempData.value.id !== null) {
    // 編輯模式
    try {
      await axios.patch(apiUrl, {
        id: tempData.value.id,
        name: tempData.value.name,
        age: Number(tempData.value.age),
      });
      await fetchData();
      resetForm();
    } catch (error) {
      console.error('Error updating data:', error);
    }
  } else {
    // 新增模式
    try {
      await axios.post(apiUrl, {
        name: tempData.value.name,
        age: Number(tempData.value.age),
      });
      await fetchData();
      resetForm();
    } catch (error) {
      console.error('Error adding data:', error);
    }
  }
}

async function handleClickOption(btn: btnType, data: DataItemWithEditing) {
  if (btn.status === 'edit') {
    console.log('data', data);
    // 將選中行的數據填充到 tempData 中
    tempData.value = {
      id: data.id,
      name: data.name,
      age: data.age.toString(),
    };
  } else if (btn.status === 'delete') {
    $q.dialog({
      title: '提示',
      message: '是否確定刪除該筆資料？',
      cancel: true,
      persistent: true,
    }).onOk(async () => {
      try {
        await axios.delete(`${apiUrl}/${data.id}`);
        await fetchData();
      } catch (error) {
        console.error('刪除數據時出錯:', error);
      }
    });
  }
}

function resetForm() {
  tempData.value = { id: null, name: '_', age: '1' };
}
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
