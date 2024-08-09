<template>
  <div class="layout-padding">
    <div class="layout-padding-auto layout-padding-view">
      <el-button @click="dialogVisible = true">点击绘制图表</el-button>
        <div class="sm:flex"></div>
          <el-dialog title="图表" 
            :modal-append-to-body='true'
            v-model="dialogVisible"
              @open="makeChart1(tableName, y1Name, y2Name, num)"
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
            <!-- <el-select placeholder="选择x轴参数" v-model="xName">
              <el-option :label="'createTime'" :value="column.list[1]"></el-option>
              <el-option :label="'updateTime'" :value="column.list[5]"></el-option>
            </el-select> -->
            <el-select placeholder="选择一个y轴参数" v-model="y1Name">
              <el-option :key="item" :label="item" :value="item" v-for="item in column.list"></el-option>
            </el-select>
            <el-select placeholder="选择一个y轴参数" v-model="y2Name">
              <el-option :key="item" :label="item" :value="item" v-for="item in column.list"></el-option>
            </el-select>
            <div class="demo-datetime-picker">
              <div class="block">
                <span class="demonstration">StartTime</span>
                <el-date-picker
                  v-model="start"
                  type="datetime"
                  placeholder="Select date and time"
                />
              </div>
              <div class="block">
                <span class="demonstration">EndTime</span>
                <el-date-picker
                  v-model="end"
                  type="datetime"
                  placeholder="Select date and time"
                />
              </div>
            </div>
            <el-input-number v-model="num" :min="-10" :max="10" @change="handleChange" />
            <!-- <el-select placeholder="选择起始时间" v-model="vary3">
              <el-option :key="item" :label="item" :value="item" v-for="item in time.list"></el-option>
            </el-select>
            <el-select placeholder="选择终止时间" v-model="vary4">
              <el-option :key="item" :label="item" :value="item" v-for="item in time.list"></el-option>
            </el-select> -->
          </el-form-item>
          <!-- <el-button icon="folder-add" type="primary" class="ml10" @click="formDialogRef.openDialog()"
            v-auth="'demo_demo_add'">
            新 增
          </el-button>
          <el-button plain :disabled="multiple" icon="Delete" type="primary"
            v-auth="'demo_demo_del'" @click="handleDelete(selectObjs)">
            删除
          </el-button> -->
          <!-- <el-button icon="folder-add" type="primary" class="ml10" @click="() => makeChart('username', 'nicename')"> -->
          <!-- <el-button icon="folder-add" type="primary" class="ml10" @click="() => makeChart1(tableName, vary1, vary2, vary3, vary4)">
            绘图
          </el-button> -->
          <!-- <right-toolbar v-model:showSearch="showSearch" :export="'demo_demo_export'"
                @exportExcel="exportExcel" class="ml10 mr20" style="float: right;"
            @queryTable="getDataList"></right-toolbar> -->
        </div>
      </el-row>
      <!-- <el-table :data="state.dataList" v-loading="state.loading" border 
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
      <div class="sm:flex"> -->
		</div>

    <!-- 编辑、新增  -->
    <form-dialog ref="formDialogRef" @refresh="getDataList(false)" />
  </div>
</template>

<script setup lang="ts" name="systemDemo">
import dayjs from 'dayjs';
import { BasicTableProps, useTable } from "/@/hooks/table";
import { fetchList, delObjs , getObjlist , getObjlist_page, getColumn, getProcess , getTimeList} from "/@/api/demo/demo";
import { useMessage, useMessageBox } from "/@/hooks/message";
import { useDict } from '/@/hooks/dict';
import * as echarts from 'echarts';
import { systemCache } from '/@/api/admin/system';
import { markRaw } from 'vue';
import { useRoute } from 'vue-router'


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


const route = useRoute()

let tableName = route.query.tableName;

console.log(tableName)

const getList = async () => {
  column.list = await getColumn({tableName:tableName})
  // console.log(column.list)
}

getList()

// const time = reactive({
//   list: <any>[]
// })

// const getTime = async () => {
//   time.list = await getTimeList({tableName:tableName})
//   // console.log(time.list)
// }

// getTime()

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

const xName = ref();
const y1Name = ref();
const y2Name = ref();
const start = ref('');
const end = ref('');

const num = ref(1)
const handleChange = (value: number) => {
  console.log(value)
}

const dialogVisible = ref(false);

// //下拉菜单选择时间
// function change(varx: any) {
//   let var1 = '';
//   if (varx == 'create_time') {
//     var1 = 'createTime';
//   } else if (varx == 'update_time') {
//     var1 = 'updateTime';
//   }
//   return var1;
// }

//制表


const commandChartRef = ref();

const chartOptions = reactive({
  commandChartOption: {
    legend: {
        data: ['first', 'second', ]
    },
    xAxis: {
      type: 'time',
      axisLabel: {
        //formatter: '{yyyy}-{MM}-{dd} {HH}:{mm}:{ss}'
        formatter: '{HH}:{mm}:{ss}',
        rotate: 40
      }
    },
    yAxis: {},
		series: [
      {
        name: 'first',
        type: 'line',
        data: <any>[[]],
        lineStyle: {
            color: 'green',
            width: 4,
            type: 'solid'
        }
      },
      {
        name: 'second',
        type: 'line',
        data: <any>[[]],
        lineStyle: {
            color: 'blue',
            width: 4,
            type: 'solid'
        }
			},
    ],
    tooltip: { // 鼠标悬浮提示框显示 X和Y 轴数据
     trigger: 'item',
     backgroundColor: 'rgba(32, 33, 36,.7)',
     borderColor: 'rgba(32, 33, 36,0.20)',
     borderWidth: 1,
     textStyle: { // 文字提示样式
       color: '#fff',
       fontSize: '12'
     },
     axisPointer: { // 坐标轴虚线
       type: 'cross',
       label: {
           backgroundColor: '#6a7985'
       }
     },
    }  
	},
});

// const alpha = 0.5;
// //指数滤波
// function exponentialSmoothing(data:any, alpha:any) {
//   let smoothedData = [data[0]]; // 初始化平滑后的数据数组

//   for (let i = 1; i < data.length; i++) {
//     const smoothedValue = alpha * data[i] + (1 - alpha) * smoothedData[i - 1];
//     smoothedData.push(smoothedValue);
//   }

//   return smoothedData;
// }

//滑动平均滤波制图

// console.log(vary3.value)

const makeChart1 = async (property1: any, property2: any, property3: any, property4:any) => {

  const dateObj1 = new Date(start.value);
  const starting = dayjs(dateObj1).format('YYYY-MM-DD HH:mm:ss');
  const dateObj2 = new Date(end.value);
  const ending = dayjs(dateObj2).format('YYYY-MM-DD HH:mm:ss');
  // console.log(my)
 
  const detaaxis1 = await getProcess({ tableName: property1, startTime: starting, endTime: ending, columnName: property2 , num: 0})
  const detaaxis2 = await getProcess({ tableName: property1, startTime: starting, endTime: ending , columnName: property3 , num: property4})
  const timeaxis = await getObjlist({ tableName: property1, startTime : starting, endTime : ending })
  //let result1 = res1.map((item) => item[change(property1)]);

  let pairs1 = timeaxis.map((item, index) => [item, detaaxis1[index]]);
  let pairs2 = timeaxis.map((item, index) => [item, detaaxis2[index]]);

  chartOptions.commandChartOption.series[0].data = pairs1;
  chartOptions.commandChartOption.series[1].data = pairs2;
  //console.log(chartOptions.commandChartOption.xAxis.data)

  const commandChart = markRaw(echarts.init(commandChartRef.value));
  commandChart.setOption(chartOptions.commandChartOption);
};

// //指数滤波制图
// const makeChart2 = async (property1: any, property2: any, property3: any) => {
//   const res = await getObjlist()

//   // let result1 = res.map((item) => new Date(item[change(property1)]));
//   let result1 = res.map((item) => item[change(property1)]);
//   let result2 = res.map((item) => item[property2]);
//   let pairs1 = result1.map((item, index) => [item, result2[index]]);
//   let result3 = res.map((item) => item[property3]);
//   let pairs2 = result1.map((item, index) => [item, result3[index]]);

//   let arr2 = new Array();
//   let arr3 = new Array();

//   arr2 = exponentialSmoothing(result2, alpha)
//   arr3=exponentialSmoothing(result3, alpha)

//   // chartOptions.commandChartOption.xAxis.data = result1;
//   // console.log(chartOptions.commandChartOption.xAxis.data)
//   chartOptions.commandChartOption.series[0].data = pairs1;
//   console.log(chartOptions.commandChartOption.series[0].data)
//   chartOptions.commandChartOption.series[1].data = pairs2;

//   const commandChart = markRaw(echarts.init(commandChartRef.value));
//   commandChart.setOption(chartOptions.commandChartOption);
// };
</script>

<style scoped>
.demo-datetime-picker {
  display: flex;
  width: 100%;
  padding: 0;
  flex-wrap: wrap;
}
.demo-datetime-picker .block {
  padding: 30px 0;
  text-align: center;
  border-right: solid 1px var(--el-border-color);
  flex: 1;
}
.demo-datetime-picker .block:last-child {
  border-right: none;
}
.demo-datetime-picker .demonstration {
  display: block;
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 20px;
}
</style>