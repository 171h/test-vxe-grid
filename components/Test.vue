<script setup lang="ts">
import { defineComponent, ref, reactive } from "vue";
import { VxeGridProps, VxeTableInstance, VXETable, VxeToolbarInstance, VxeTablePropTypes, VxeTableEvents } from "vxe-table";
import * as _ from "lodash-es";
import { Logger } from "@171h/log";

/**
 * 测试功能：
 * 1. 数据的增删改查
 * 2. 数据的底部添加空行
 * 3. 数据的底部空行的滚动加载与删除
 * 4. 编辑渲染器的 v-model.lazy 的使用, 待解决阻止 textarea enter 换行问题 【基本解决】
 * 5. 数据的扁平化导出功能
 * 6. cell 的上下左右边距过宽的问题【已解决】
 * 7. 解决 tree 树形表格的单元格 edit 状态无法撑满 td 的问题【已解决】
 *
 */

const logger = new Logger("test.vue");

interface RowVO {
  id: number;
  parentId: number | null;
  name: string;
  type: string;
  size: number | null;
  date: string;
}

const data: RowVO[] = [
  { id: 10000, parentId: null, name: "test abc1", type: "mp3", size: 1024, date: "2020-08-01" },
  { id: 10050, parentId: null, name: "Test2", type: "mp4", size: null, date: "2021-04-01" },
  { id: 24300, parentId: 10050, name: "Test3", type: "avi", size: 1024, date: "2020-03-01" },
  { id: 20045, parentId: 24300, name: "test abc4", type: "html", size: 600, date: "2021-04-01" },
  { id: 10053, parentId: 24300, name: "test abc96", type: "avi", size: null, date: "2021-04-01" },
  { id: 24330, parentId: 10053, name: "test abc5", type: "txt", size: 25, date: "2021-10-01" },
  { id: 21011, parentId: 10053, name: "Test6", type: "pdf", size: 512, date: "2020-01-01" },
  { id: 22200, parentId: 10053, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 23666, parentId: null, name: "Test8", type: "xlsx", size: 2048, date: "2020-11-01" },
  { id: 23677, parentId: 23666, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 23671, parentId: 23677, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 23672, parentId: 23677, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 23688, parentId: 23666, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 23681, parentId: 23688, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 23682, parentId: 23688, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 24555, parentId: null, name: "test abc9", type: "avi", size: 224, date: "2020-10-01" },
  { id: 24566, parentId: 24555, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
  { id: 24577, parentId: 24555, name: "Test7", type: "js", size: 1024, date: "2021-06-01" },
];

const demo1 = reactive({
  value400: "",
  value407: "",
  value408: "2020-10-01",
  value409: "2020-10-01",
  value800: "",
  value801: "",
  value802: "",
  value803: "",
  value804: "",
  value805: "",
});

const data2 = ref(data);

// 工具栏绑定===========================================>>>>>>>>>>>>>>>>>>>>>>>>>>>>
const xGrid = ref<VxeTableInstance>();
const xToolbar = ref<VxeToolbarInstance>();
nextTick(() => {
  const $grid = xGrid.value;
  const $toolbar = xToolbar.value;
  if ($grid && $toolbar) {
    $grid.connect($toolbar);
  }
});

// ===========================================>>>>>>>>>>>>>>>>>>>>>>>>>>> gridOptions
const gridOptions = reactive<VxeGridProps<RowVO>>({
  border: true,
  columns: [
    { type: "checkbox", width: 50, showOverflow: true, align: "center" },
    { type: "seq", width: 100 },
    { field: "name", title: "Name", slots: { edit: "text_edit" }, treeNode: true, editRender: { enabled: true } },
    { field: "type", title: "Type", slots: { edit: "text_edit" }, showOverflow: true, editRender: { enabled: true } },
    {
      field: "size",
      title: "Size",
      slots: { edit: "text_edit" },
      showHeaderOverflow: true,
      editRender: { enabled: true },
    },
    { field: "date", title: "Date", slots: { edit: "text_edit" }, showOverflow: true, editRender: { enabled: true } },
    { title: "操作", slots: { default: "action" }, width: 400, editRender: { enabled: true } },
  ],
  treeConfig: {
    transform: true,
    rowField: "id",
    parentField: "parentId",
    expandAll: true,
  },
  loading: false,
  rowConfig: {
    keyField: "id",
    isHover: true,
  },
  checkboxConfig: {
    checkRowKeys: [10001, 10002],
    highlight: true,
  },
  data: data2.value,
  editConfig: {
    trigger: "click",
    mode: "cell",
    showStatus: true,
  },
});

// 读取文件===========================================>>>>>>>>>>>>>>>>>>>>>>>>>>>>
const clickEvent1 = async () => {
  try {
    const { file } = await VXETable.readFile();
    logger.info(file);
    VXETable.modal.alert(`文件名：${file.name}，文件大小：${file.size}`);
  } catch (e) {}
};

// 打印===========================================>>>>>>>>>>>>>>>>>>>>>>>>>>>>
// 打印样式
const printStyle = `
.title {
  text-align: center;
}
.my-list-row {
  display: inline-block;
  width: 100%;
}
.my-list-col {
  float: left;
  width: 33.33%;
  height: 28px;
  line-height: 28px;
}
.my-top,
.my-bottom {
  font-size: 12px;
}
.my-top {
  margin-bottom: 5px;
}
.my-bottom {
  margin-top: 30px;
  text-align: right;
}
`;

// 打印顶部内容模板
const topHtml = `
<h1 class="title">出货单据</h1>
<div class="my-top">
  <div class="my-list-row">
    <div class="my-list-col">商品名称：vxe-table</div>
    <div class="my-list-col">发货单号：X2665847132654</div>
    <div class="my-list-col">发货日期：2020-09-20</div>
  </div>
  <div class="my-list-row">
    <div class="my-list-col">收货姓名：小徐</div>
    <div class="my-list-col">收货地址：火星第七区18号001</div>
    <div class="my-list-col">联系电话：10086</div>
  </div>
</div>
`;
// 打印底部内容模板
const bottomHtml = `
<div class="my-bottom">
  <div class="my-list-row">
    <div class="my-list-col"></div>
    <div class="my-list-col">创建人：小徐</div>
    <div class="my-list-col">创建日期：2020-09-20</div>
  </div>
</div>
`;
const printEvent1 = () => {
  const $grid = xGrid.value;
  $grid?.print({
    sheetName: "打印出货单据",
    style: printStyle,
    columns: [{ type: "seq" }, { field: "name" }, { field: "role" }, { field: "address" }],
    beforePrintMethod: ({ content }) => {
      // 拦截打印之前，返回自定义的 html 内容
      return topHtml + content + bottomHtml;
    },
  });
};

// 增删改查===========================================>>>>>>>>>>>>>>>>>>>>>>>>>>>>
// 增加行
const insertRowEnd = async (num: number) => {
  const $table = xGrid.value!;
  for (let i = 0; i < num; i++) {
    const date = new Date();
    const record = {
      name: `新数据 ${i + 1}`,
      id: Date.now(),
      parentId: null,
      date: `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`,
    };
    const { row: newRow } = await $table.insertAt(record, -1);
  }
};

const insertRow = async (currRow: any, locat: string) => {
  const $table = xGrid.value!;
  const date = new Date();

  // 如果 null 则插入到目标节点顶部
  // 如果 -1 则插入到目标节点底部
  // 如果 row 则有插入到效的目标节点该行的位置

  // 同级节点，上一行插入
  if (locat === "current") {
    const record = {
      name: "新数据",
      id: Date.now(),
      parentId: currRow.parentId, // 父节点必须与当前行一致
      date: `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`,
    };
    const { row: newRow } = await $table.insertAt(record, currRow);
    await $table.setEditRow(newRow); // 插入子节点
  }
  // 子节点，下一行插入
  else if (locat === "top") {
    const record = {
      name: "新数据",
      id: Date.now(),
      parentId: currRow.id, // 需要指定父节点，自动插入该节点中
      date: `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`,
    };
    const { row: newRow } = await $table.insert(record);
    await $table.setTreeExpand(currRow, true); // 将父节点展开
    await $table.setEditRow(newRow); // 插入子节点
  }
  // 子节点，子节点最后一行的下一行插入
  else if (locat === "bottom") {
    const record = {
      name: "新数据",
      id: Date.now(),
      parentId: currRow.id, // 需要指定父节点，自动插入该节点中
      date: `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`,
    };
    const { row: newRow } = await $table.insertAt(record, -1);
    await $table.setTreeExpand(currRow, true); // 将父节点展开
    await $table.setEditRow(newRow); // 插入子节点
  }
};

// 删除行
const removeRow = async (row: any) => {
  const $table = xGrid.value!;
  await $table.remove(row);
};

// 禁用 enter
// function checkEnter(e: KeyboardEvent) {
//   var et = e || window.event;
//   var keycode = et.charCode || et.keyCode;
//   if (keycode == 13) {
//     if (window.event) {
//       window.event.returnValue = false;
//     } else {
//       e.preventDefault(); //for firefox
//     }
//   }
// }

/* 解决 tree 树形表格的单元格 edit 状态无法撑满 td 的问题 */
const selectCurrent = reactive({
  selectRow: null as any,
  selectColumn: null as any,
});
const cellClassName: VxeTablePropTypes.CellClassName = ({ row, column }) => {
  if (row === selectCurrent.selectRow && column === selectCurrent.selectColumn) {
    logger.debug("cellClassName", row, column);
    return "no-indent";
  }
  return null;
};
const cellClickEvent: VxeTableEvents.CellClick = ({ row, column }) => {
  selectCurrent.selectRow = row;
  selectCurrent.selectColumn = column;
};
</script>

<template>
  <div>
    <h1>Test</h1>
    <!-- <vxe-button @click="add">add</vxe-button>
    <vxe-button @click="remove">remove</vxe-button>
    <vxe-button @click="modify(10001)">modify</vxe-button> -->
    <vxe-button @click="clickEvent1">读取文件</vxe-button>
    <vxe-button @click="printEvent1">打印</vxe-button>
    <vxe-button @click="insertRowEnd(10)">在尾部插入空白行</vxe-button>
  </div>
  <div>
    <i class="vxe-icon-alipay roll"></i>
  </div>
  <div>
    <vxe-input v-model="demo1.value800" placeholder="日期多选" type="date" clearable multiple></vxe-input>
    <vxe-input v-model="demo1.value805" placeholder="日期和时间多选" type="datetime" clearable multiple></vxe-input>
  </div>
  <vxe-toolbar ref="xToolbar" custom>
    <template #buttons>
      <vxe-button content="自定义模板"></vxe-button>
      <vxe-button content="按钮2"></vxe-button>
      <vxe-button content="按钮3"></vxe-button>
      <vxe-button content="下拉按钮">
        <template #dropdowns>
          <vxe-button content="按钮1"></vxe-button>
          <vxe-button content="按钮2"></vxe-button>
          <vxe-button content="按钮3"></vxe-button>
        </template>
      </vxe-button>
    </template>
    <template #tools>
      <vxe-button type="text" icon="vxe-icon-question-circle-fill" class="tool-btn"></vxe-button>
      <vxe-button type="text" icon="vxe-icon-funnel" class="tool-btn" @click=""></vxe-button>
    </template>
  </vxe-toolbar>
  <vxe-grid ref="xGrid" :cell-class-name="cellClassName" @cell-click="cellClickEvent" v-bind="gridOptions">
    <template #text="{ row }">
      <span>{{ row.name }}</span>
    </template>
    <template #text_edit="{ row }">
      <!-- <vxe-input v-model="row.name"></vxe-input> -->
      <!-- 解决 vxe-inpu 无法 v-mode.lazy 的问题-->
      <!-- <input class="vxe-input--inner" v-model.lazy="row.name" /> -->
      <textarea
        class="vxe-input--inner"
        style="overflow: visible; resize: none; border-radius: 0; border: none; padding: 0;"
        :cols="5"
        :spellcheck="false"
        @keyup.enter.preventDefault()
        @keypress.enter.preventDefault()
        @keydown.enter.preventDefault()
        wrap="hard"
        v-model.lazy="row.name"
      ></textarea>
    </template>
    <template #action="{ row }">
      <vxe-button type="text" status="primary" @click="insertRow(row, 'current')">插入节点</vxe-button>
      <vxe-button type="text" status="primary" @click="insertRow(row, 'top')">顶部插入节点</vxe-button>
      <vxe-button type="text" status="primary" @click="insertRow(row, 'bottom')">尾部插入子节点</vxe-button>
      <vxe-button type="text" status="primary" @click="removeRow(row)">删除节点</vxe-button>
    </template>
  </vxe-grid>
</template>

<style>
/* 默认单元格设置 */
/* 解决 cell 的上下左右边距过宽的问题 */
.vxe-table--render-default .vxe-cell {
  padding: 0 0;
}

/* 解决 tree 树形表格的单元格 edit 状态无法撑满 td 的问题 */
.vxe-table .vxe-body--column.no-indent .vxe-cell--tree-node {
  padding-left: 0!important;
}
.vxe-table .vxe-body--column.no-indent .vxe-tree-cell {
  padding-left: 0!important;
}
.vxe-table .vxe-body--column.no-indent .vxe-tree--btn-wrapper {
  display: none;
}

</style>
