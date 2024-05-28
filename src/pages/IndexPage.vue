<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="addList()">{{
          tempData.id.length ? '修改' : '新增'
        }}</q-btn>
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
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref } from 'vue';
import { useQuasar } from 'quasar';

const $q = useQuasar();

// console.log($q.dialog);

const api = axios.create({
  baseURL: 'https://dahua.metcfire.com.tw/api/CRUDTest',
});

interface IbtnType {
  label: string;
  icon: string;
  status: string;
}

interface IdataType {
  id: string;
  name: string;
  age: string;
}

/** 列表資料 */
const blockData = ref<IdataType[]>([]);

/** 表格設定 */
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
    sortable: true,
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
    sortable: true,
  },
]);

/** 表格按鈕 */
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

/** 輸入框綁定 */
const tempData = ref({
  id: '',
  name: '',
  age: '',
});

/**
 * 更新或刪除
 * @param btn 點擊的按鈕資訊
 * @param data 該行資訊
 */
async function handleClickOption(btn: IbtnType, data: IdataType) {
  // console.log(btn, data);

  const idx = blockData.value.findIndex((item) => item.id === data.id);
  if (idx === -1) return;

  // 刪除
  if (btn.status === 'delete') {
    $q.dialog({
      title: '提示',
      message: '是否確定刪除該筆資料?',
      cancel: true,
      persistent: true,
    })
      .onOk(async () => {
        await api.delete(`/${data.id}`);
        getList();
      })
      .onOk(() => {
        // console.log('>>>> second OK catcher')
      })
      .onCancel(() => {
        // console.log('>>>> Cancel')
      })
      .onDismiss(() => {
        // console.log('I am triggered on both OK and Cancel')
      });
  }
}

/** 新增 */
async function addList() {
  // console.log(tempData.value);

  if (
    isNaN(+tempData.value.age) ||
    +tempData.value.age <= 0 ||
    !Number.isInteger(+tempData.value.age)
  ) {
    return;
  }

  if (tempData.value.name.length === 0) {
    return;
  }

  if (tempData.value.id.length > 0) {
    await api.patch('/', tempData.value);
  } else {
    await api.post('/', tempData.value);
  }
  tempData.value = {
    id: '',
    name: '',
    age: '',
  };
  getList();
}

/** 取列表 */
async function getList() {
  const { data }: { data: IdataType[] } = await api.get('/a');
  console.log(data);
  blockData.value = data;
  console.log(blockData.value);
}
getList();
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
