<script setup lang="ts">
import {
  Breadcrumb,
  BreadcrumbItem,
  Flex,
  Form,
  Table,
  TableProps,
} from 'ant-design-vue';
import { placementType } from 'ant-design-vue/es/drawer';
import axios from 'axios';
import { PrimaryButton, SearchInput } from 'src/components';
import { useSearch } from 'src/composable';
import { Dialog } from 'src/plugins';
import { computed, reactive, ref } from 'vue';
import { usePagination } from 'vue-request';
import CreateDialog from './CreateDeveloperDrawer.vue';

const loading = ref(false);
const { searchKey } = useSearch();
const formValue = reactive<{
  user: { name: string };
}>({
  user: {
    name: 'abc',
  },
});

type APIParams = {
  results: number;
  page?: number;
  sortField?: string;
  sortOrder?: number;
  [key: string]: any;
};
type APIResult = {
  results: {
    gender: 'female' | 'male';
    name: string;
    email: string;
  }[];
};

const columns = [
  {
    title: 'Action',
    dataIndex: 'action',
    width: '10%',
  },
  {
    title: 'Developer Name',
    dataIndex: 'developerName',
    sorter: true,
    /* filters: [
      { text: 'KSL', value: 'ksl' },
      { text: 'Setia', value: 'setia' },
    ], */
    width: '22.5%',
  },
  {
    title: 'Project Amount',
    dataIndex: 'developerName',
    sorter: true,
    width: '22.5%',
  },
  {
    title: 'Total Comm Disbursed',
    dataIndex: 'developerName',
    sorter: true,
    width: '22.5%',
  },
  {
    title: 'Total Comm to be Disbursed',
    dataIndex: 'developerName',
    sorter: true,
    width: '22.5%',
  },
];

const queryData = (params: APIParams) => {
  return axios.get<APIResult>('https://randomuser.me/api?noinfo', { params });
};

const {
  data: dataSource,
  run,
  current,
  pageSize,
} = usePagination(queryData, {
  formatResult: (res) => res.data.results,
  pagination: {
    currentKey: 'page',
    pageSizeKey: 'results',
  },
});

const pagination = computed(() => ({
  total: 200,
  current: current.value,
  pageSize: pageSize.value,
}));

const handleTableChange: TableProps['onChange'] = (
  pag: { pageSize: number; current: number },
  filters: any,
  sorter: any
) => {
  run({
    results: pag.pageSize,
    page: pag?.current,
    sortField: sorter.field,
    sortOrder: sorter.order,
    ...filters,
  });
};

async function showDrawer(position: placementType) {
  Dialog.create({
    title: 'Create Developer',
    component: CreateDialog,
    width: 800,
    bodyStyle: {},
    headerStyle: {
      borderTopLeftRadius: '25px',
      backgroundColor: '#FF9D2D',
      textAlign: 'center',
      color: '#FFF !important',
    },
    style: { borderTopLeftRadius: '25px' },
    componentProps: { title: 'test' },
    placement: position,
    async onOK(v) {
      await new Promise((resolve) => setTimeout(resolve, 3000));
      // console.log('abc');
      // console.log(v);
    },
  });
  // await new Promise((resolve) => setTimeout(resolve, 500));
}

function handleFinish() {
  console.log('state ', formValue);
}
</script>

<template>
  <Breadcrumb style="margin: 16px 0">
    <BreadcrumbItem>Home</BreadcrumbItem>
    <BreadcrumbItem>Developer</BreadcrumbItem>
  </Breadcrumb>

  <div style="text-align: left">
    <p style="color: black; line-height: normal; margin: 5px">Search</p>
    <Flex justify="space-between">
      <Form :model="formValue" @finish="handleFinish">
        <SearchInput
          v-model="searchKey"
          style="margin-bottom: 12px; width: 250px"
        />
      </Form>
      <PrimaryButton
        btn-class="abc bg-fprimary-1"
        :loading="loading"
        style="line-height: normal"
        @click="showDrawer('right')"
      >
        Create Developer
      </PrimaryButton>
    </Flex>
  </div>

  <div>
    <Table
      :columns="columns"
      :row-key="(record) => record.login.uuid"
      :data-source="dataSource"
      :pagination="pagination"
      :loading="loading"
      @change="handleTableChange"
    >
      <template #bodyCell="{ column, text }">
        <template v-if="column.dataIndex === 'name'">
          {{ text.first }} {{ text.last }}
        </template>
      </template>
    </Table>
  </div>
</template>
<style lang="sass">
.ant-drawer-title
  color: white !important
</style>
