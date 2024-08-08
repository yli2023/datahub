<template>
  <div class="layout-padding">
    <div class="layout-padding-auto layout-padding-view">
      <el-button @click="dialogVisible = true">点击绘制图表</el-button>
        <div class="sm:flex"></div>
          <el-dialog title="图表" 
            :modal-append-to-body='true'
            v-model="dialogVisible"
              @open="makeChart1(varx, vary1, vary2, vary3, vary4)"
              append-to-body>
              <el-card class="sm:mr-4 flex-1 !border-none mt-4" shadow="never">
                <div>
                  <div class="flex h-[400px] items-center" ref="commandChartRef" style="flex-grow: 2;"></div>
                </div>
              </el-card>        
          </el-dialog>
      <el-row>
        <div class="mb8" style="width: 100%">
          <el-form-item label="参数选择">
            <el-select placeholder="选择x轴参数" v-model="varx">
              <el-option :label="'createTime'" :value="column.list[1]"></el-option>
              <el-option :label="'updateTime'" :value="column.list[5]"></el-option>
            </el-select>
            <el-select placeholder="选择一个y轴参数" v-model="vary1">
              <el-option :key="item" :label="item" :value="item" v-for="item in column.list"></el-option>
            </el-select>
            <el-select placeholder="选择一个y轴参数可为空" v-model="vary2">
              <el-option :key="item" :label="item" :value="item" v-for="item in column.list"></el-option>
            </el-select>
            <el-select placeholder="选择起始时间" v-model="vary3">
              <el-option :key="item" :label="item" :value="item" v-for="item in time.list"></el-option>
            </el-select>
            <el-select placeholder="选择终止时间" v-model="vary4">
              <el-option :key="item" :label="item" :value="item" v-for="item in time.list"></el-option>
            </el-select>
          </el-form-item>
          <el-button icon="folder-add" type="primary" class="ml10" @click="formDialogRef.openDialog()"
            v-auth="'demo_demo_add'">
            新 增
          </el-button>
          <el-button plain :disabled="multiple" icon="Delete" type="primary"
            v-auth="'demo_demo_del'" @click="handleDelete(selectObjs)">
            删除
          </el-button>
          <!-- <el-button icon="folder-add" type="primary" class="ml10" @click="() => makeChart('username', 'nicename')"> -->
          <el-button icon="folder-add" type="primary" class="ml10" @click="() => makeChart1(varx, vary1, vary2, vary3, vary4)">
            绘图
          </el-button>
          <right-toolbar v-model:showSearch="showSearch" :export="'demo_demo_export'"
                @exportExcel="exportExcel" class="ml10 mr20" style="float: right;"
            @queryTable="getDataList"></right-toolbar>
        </div>
      </el-row>
      <el-table :data="state.dataList" v-loading="state.loading" border 
        :cell-style="tableStyle.cellStyle" :header-cell-style="tableStyle.headerCellStyle"
				@selection-change="selectionChangHandle"
        @sort-change="sortChangeHandle">
        <el-table-column type="selection" width="40" align="center" />
        <el-table-column type="index" label="#" width="40" />
          <el-table-column prop="username" label="用户名"  show-overflow-tooltip/>
          <el-table-column prop="nicename" label="昵称"  show-overflow-tooltip/>
        <el-table-column label="操作" width="150">
          <template #default="scope">
            <el-button icon="edit-pen" text type="primary" v-auth="'demo_demo_edit'"
              @click="formDialogRef.openDialog(scope.row.id)">编辑</el-button>
            <el-button icon="delete" text type="primary" v-auth="'demo_demo_del'" @click="handleDelete([scope.row.id])">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" v-bind="state.pagination" />
      <div class="sm:flex">
		</div>
    </div>

    <!-- 编辑、新增  -->
    <form-dialog ref="formDialogRef" @refresh="getDataList(false)" />


  </div>
</template>

<script setup lang="ts" name="systemDemo">
// import { defineAsyncComponent } from 'vue';
import { BasicTableProps, useTable } from "/@/hooks/table";
import { fetchList, delObjs , getObjlist , getObjlist_page, getColumn, getProcess , getTimeList} from "/@/api/demo/demo";
import { useMessage, useMessageBox } from "/@/hooks/message";
import { useDict } from '/@/hooks/dict';
import * as echarts from 'echarts';
import { systemCache } from '/@/api/admin/system';
import { markRaw } from 'vue';
import moment from 'moment'


// 引入组件
const FormDialog = defineAsyncComponent(() => import('./form.vue'));
// 定义查询字典

// 定义变量内容
const formDialogRef = ref()
// 搜索变量
const queryRef = ref()
const showSearch = ref(true)
// 多选变量
const selectObjs = ref([]) as any
const multiple = ref(true)

const state: BasicTableProps = reactive<BasicTableProps>({
  queryForm: {},
  pageList: fetchList
})

const column = reactive({
  list: <any>[]
})

const getList = async () => {
  column.list = await getColumn()
  console.log(column.list)
}

getList()

const time = reactive({
  list: <any>[]
})

const getTime = async () => {
  time.list = await getTimeList()
  // console.log(time.list)
}

getTime()

// const state: BasicTableProps = reactive<BasicTableProps>({
//   queryForm: {},
//   pageList: getDataList
// })

//  table hook
const {
  getDataList,
  currentChangeHandle,
  sizeChangeHandle,
  sortChangeHandle,
  downBlobFile,
	tableStyle
} = useTable(state)


// 筛选操作
const handleSelectList = async () => {
  state.queryForm = {};
  state.pageList = getObjlist_page
  getDataList()
  // getObjlist
  // console.log(state.pageList)
  // const state: BasicTableProps = reactive<BasicTableProps>({
  //   queryForm: {},
  //   pageList: getObjlist
  // });

  // try {
  //   state.queryForm = {"id": 1}
	// 	await getDataList();
	// 	// getDataList();
	// } catch (err: any) {
	// 	useMessage().error(err.msg);
	// }
};

// 清空搜索条件
const resetQuery = () => {
  // 清空搜索条件
  queryRef.value?.resetFields()
  // 清空多选
  selectObjs.value = []
  getDataList()
}

// 导出excel
const exportExcel = () => {
  downBlobFile('/demo/demo/export', Object.assign(state.queryForm, { ids: selectObjs }), 'demo.xlsx')
}

// 多选事件
const selectionChangHandle = (objs: { id: string }[]) => {
  selectObjs.value = objs.map(({ id }) => id);
  multiple.value = !objs.length;
};

// 删除操作
const handleDelete = async (ids: string[]) => {
  try {
    await useMessageBox().confirm('此操作将永久删除');
  } catch {
    return;
  }

  try {
    await delObjs(ids);
    getDataList();
    useMessage().success('删除成功');
  } catch (err: any) {
    useMessage().error(err.msg);
  }
};

const varx = ref();
const vary1 = ref();
const vary2 = ref();
const vary3 = ref();
const vary4 = ref();

const dialogVisible = ref(false);

const openDialog = () => {
  dialogVisible.value = true;
};

//下拉菜单选择时间
function change(varx: any) {
  let var1 = '';
  if (varx == 'create_time') {
    var1 = 'createTime';
  } else if (varx == 'update_time') {
    var1 = 'updateTime';
  }
  return var1;
}

//制表
const commandChartRef = ref();

const chartOptions = reactive({
	commandChartOption: {
    xAxis: {
      type: 'time',
      axisLabel: {
        //formatter: '{yyyy}-{MM}-{dd} {HH}:{mm}:{ss}'
        formatter: '{HH}:{mm}:{ss}'
      }
    },
    yAxis: {},
		series: [
			{
        type: 'line',
        data: <any>[[]],
        lineStyle: {
            color: 'green',
            width: 4,
            type: 'dashed'
        }
      },
      {
        type: 'line',
        data: <any>[[]],
        lineStyle: {
            color: 'blue',
            width: 4,
            type: 'dashed'
        }
			},
		],
	},
});

const alpha = 0.5;
//指数滤波
function exponentialSmoothing(data:any, alpha:any) {
  let smoothedData = [data[0]]; // 初始化平滑后的数据数组

  for (let i = 1; i < data.length; i++) {
    const smoothedValue = alpha * data[i] + (1 - alpha) * smoothedData[i - 1];
    smoothedData.push(smoothedValue);
  }

  return smoothedData;
}

//滑动平均滤波制图
const makeChart1 = async (property1: any, property2: any, property3: any, property4: any, property5: any) => {
  const detaaxis1 = await getProcess({ startTime: property4, endTime: property5 , columnName: property2})
  const detaaxis2 = await getProcess({ startTime: property4, endTime: property5 , columnName: property3})
  const timeaxis = await getObjlist({ startTime : property4, endTime : property5 })
  //let result1 = res1.map((item) => item[change(property1)]);

  let pairs1 = timeaxis.map((item, index) => [item, detaaxis1[index]]);
  let pairs2 = timeaxis.map((item, index) => [item, detaaxis2[index]]);

  chartOptions.commandChartOption.series[0].data = pairs1;
  chartOptions.commandChartOption.series[1].data = pairs2;
  //console.log(chartOptions.commandChartOption.xAxis.data)

  const commandChart = markRaw(echarts.init(commandChartRef.value));
  commandChart.setOption(chartOptions.commandChartOption);
};

//指数滤波制图
const makeChart2 = async (property1: any, property2: any, property3: any) => {
  const res = await getObjlist()

  // let result1 = res.map((item) => new Date(item[change(property1)]));
  let result1 = res.map((item) => item[change(property1)]);
  let result2 = res.map((item) => item[property2]);
  let pairs1 = result1.map((item, index) => [item, result2[index]]);
  let result3 = res.map((item) => item[property3]);
  let pairs2 = result1.map((item, index) => [item, result3[index]]);

  let arr2 = new Array();
  let arr3 = new Array();

  arr2 = exponentialSmoothing(result2, alpha)
  arr3=exponentialSmoothing(result3, alpha)

  // chartOptions.commandChartOption.xAxis.data = result1;
  // console.log(chartOptions.commandChartOption.xAxis.data)
  chartOptions.commandChartOption.series[0].data = pairs1;
  console.log(chartOptions.commandChartOption.series[0].data)
  chartOptions.commandChartOption.series[1].data = pairs2;

  const commandChart = markRaw(echarts.init(commandChartRef.value));
  commandChart.setOption(chartOptions.commandChartOption);
};
</script>
