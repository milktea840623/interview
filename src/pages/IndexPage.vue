<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="handleAdd">更新</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="tableConfig"
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
    <q-dialog v-model="isEditDialogOpen">
      <EditMember
        :editMember="selectedMember"
        @update="updateMember"
        @close="isEditDialogOpen = false"
      />
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
// import { watch } from 'fs';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';
import EditMember from '../components/EditMember.vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const isEditDialogOpen = ref(false);
const selectedMember = ref({ name: '', age: 0 });
const blockData = ref([]);
const tableConfig = ref<QTableProps['columns']>([
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
]);

const tempData = ref({
  name: '',
  age: '',
});

async function handleClickOption(
  btn: btnType,
  data: { id: string; name: string; age: number }
) {
  if (btn.status === 'edit') {
    selectedMember.value = { ...data };
    isEditDialogOpen.value = true;
  } else {
    alert('確認刪除');
    const res = await axios.delete(
      `https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`
    );
    if (res.status === 200) {
      handleGet();
    }
  }
}
const updateMember = async (updatedData: {
  id: string;
  name: string;
  age: number;
}) => {
  if (
    updatedData.name === '' ||
    isNaN(Number(updatedData.age)) ||
    !Number.isInteger(Number(updatedData.age)) ||
    Number(updatedData.age) <= 0
  ) {
    alert('請輸入姓名和年齡');
    return;
  }
  const res = await axios.patch(
    `https://dahua.metcfire.com.tw/api/CRUDTest`,
    updatedData
  );
  if (res.status === 200) {
    isEditDialogOpen.value = false;
    handleGet();
  }
};

const handleAdd = async () => {
  if (
    tempData.value.name === '' ||
    isNaN(Number(tempData.value.age)) ||
    tempData.value.age === '' ||
    !Number.isInteger(Number(tempData.value.age)) ||
    Number(tempData.value.age) <= 0
  ) {
    alert('請輸入姓名和年齡');
    return;
  }
  try {
    await axios.post(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      tempData.value
    );
  } catch (error) {
    console.log(error);
  } finally {
    handleGet();
  }
};

const handleGet = async () => {
  try {
    const res = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    blockData.value = res.data;
  } catch (error) {
    console.log(error);
  } finally {
  }
};

onMounted(() => {
  handleGet();
});
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
