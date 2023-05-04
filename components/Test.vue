<script setup lang="ts">
import { defineComponent, ref, reactive } from "vue";
import {
  VxeGridProps,
  VxeTableInstance,
  VXETable,
  VxeToolbarInstance,
  VxeTablePropTypes,
  VxeTableEvents,
  VxeGridListeners,
} from "vxe-table";
import * as _ from "lodash-es";
import { Logger } from "@171h/log";
import Sortable from "sortablejs";

/**
 * 测试功能：
 * 1. 数据的增删改查 【可以实现】
 * 2. 数据的底部添加空行 【可以实现】
 * 3. 数据的底部空行的滚动加载与删除【可以实现】
 * 4. 编辑渲染器的 v-model.lazy 的使用, 待解决阻止 textarea enter 换行问题【已解决】
 * 5. 数据的扁平化导出功能 【可以实现】
 * 6. cell 的上下左右边距过宽的问题【已解决】
 * 7. 解决 tree 树形表格的单元格 edit 状态无法撑满 td 的问题【已解决】
 * 8. 实现数据流从数据源到表格的单项传递【基本实现】
 * 8.1 {@link insertRowBySource} 在任意位置插入行
 * 8.2 {@link insertRowEndBySource} 在末尾插入行
 * 8.3 {@link modifyCell} 修改单元格数据
 * 8.4 {@link removeRow} 删除行
 * 9. 单元格数值变动后，改变其背景色【已解决】{@link ShowChangeCell }
 * 10. tree 树形表格键盘导航问题【已解决】
 * 11. 行、列拖拽【可以实现】{@see https://vxetable.cn/other4/#/table/other/sortableRow, @see https://vxetable.cn/other4/#/table/other/sortableColumn}
 */

const logger = new Logger("test.vue");

// 数据源===========================================>>>>>>>>>>>>>>>>>>>>>>>>>>>>
interface RowVO {
  id: number;
  parentId: number | null;
  name?: string;
  type?: string;
  size?: number | null;
  date?: string;
}

const data = reactive<RowVO[]>([
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
]);
const version = ref<number>(0)

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

// const status = computed(() => {
//   const $xGrid = xGrid.value
//   const select = $xGrid?.getSelectedCell()
//   return select
// })   

// watchEffect(() => {
//   logger.debug('watch', 'status', status)
// })


// gridOptions===========================================>>>>>>>>>>>>>>>>>>>>>>>>>>>
const gridOptions = reactive<VxeGridProps<RowVO>>({
  border: true,
  columns: [
    { title: "", align: "left", width: 40, slots: { default: "index" } },
    { type: "checkbox", width: 50, align: "center"},
    { type: "seq", title: "WBS", width: 100, slots: { default: "text" }},
    { field: "name", title: "Name", slots: { default:'text', edit: "text_edit2" }, treeNode: true, editRender: { enabled: true } },
    { field: "type", title: "Type", slots: { default:'text_show_change', edit: "text_edit2" }, showOverflow: true, editRender: { enabled: true } },
    {
      field: "size",
      title: "Size",
      slots: { default:'text_show_change', edit: "text_edit2" },
      showHeaderOverflow: true,
      editRender: { enabled: true },
    },
    { field: "date", title: "Date", slots: { default:'text_show_change', edit: "text_edit2" }, showOverflow: true, editRender: { enabled: true } },
    { title: "操作", slots: { default: "action" }, width: 400 },
  ],
  showOverflow: true,
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
    isCurrent: true,
  },
  checkboxConfig: {
    checkRowKeys: [10001, 10002],
    highlight: true,
  },
  data: data,
  editConfig: {
    trigger: "click",
    mode: "cell",
    showStatus: true,
  },
  mouseConfig: {
    selected: true,
  },
  keyboardConfig: {
    isArrow: true,
    isDel: true,
    isEnter: true,
    isTab: true,
    isEdit: true,
    isEsc: true,
    isClip: true,
    isChecked: true,
    isShift: true, // 开启 Shift 键批量选择
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
    await $table.insertAt(record, -1);
  }
};

const insertRowEndBySource = async (num: number) => {
  new Promise((resolve) => {
    const date = new Date();
    for (let i = 0; i < num; i++) {
      const id = data[data.length - 1].id + 1;
      const record = {
        name: `新数据 ${i + 1}`,
        id: id,
        parentId: null,
        date: `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`,
      };
      data.push(record);
    }
    version.value++;
    resolve(true);
  });
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

const insertRowBySource = async (currRow: any, locat: "before" | "after" | "last", amout?: number) => {
  return new Promise((resolve) => {
    if (!amout) amout = 1;
    const date = new Date();
    const record = {
      name: "新数据",
      id: Date.now(),
      parentId: currRow.parentId, // 需要指定父节点，自动插入该节点中
      date: `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`,
    };
    const index = data.findIndex((item) => item.id === currRow.id);
    if (locat === "before") {
      for (let i = 0; i < amout; i++) {
        data.splice(index, 0, record);
      }
    } else if (locat === "after") {
      for (let i = 0; i < amout; i++) {
        data.splice(index + 1, 0, record);
      }
    } else if (locat === "last") {
      for (let i = 0; i < amout; i++) {
        data.push(record);
      }
    }
    version.value++;
    resolve(data);
  });
};

const modifyCell = async (rowId: number, columnField: "name" | "type" | "size", value: any) => {
  return new Promise((resolve) => {
    const row = data.find((item) => item.id === rowId);
    if (row) {
      row[columnField] = value;
    }
    version.value++;
    logger.debug(modifyCell, 'version.value', version.value)
    resolve(row);
  });
};

// 删除行
const removeRow = async (...rowId: number[]) => {
  return new Promise((resolve) => {
    for (let i = 0; i < rowId.length; i++) {
      const index = data.findIndex((item) => item.id === rowId[i]);
      if (index !== -1) {
        data.splice(index, 1);
      }
    }
    resolve(data);
  });
};

// 获取当前选中的单元格
// const selectCurrent = reactive({
//   selectRow: null as any,
//   selectColumn: null as any,
// });
// const cellClickEvent: VxeTableEvents.CellClick = async (params) => {
//   new Promise((resolve) => {
//     const { row, column } = params;
//     selectCurrent.selectRow = row;
//     selectCurrent.selectColumn = column;
//     const $grid = xGrid.value;
//     logger.debug(cellClickEvent, "getRowIndex", $grid?.getRowIndex(row));
//     logger.debug(cellClickEvent, "getVTRowIndex", $grid?.getVTRowIndex(row));
//     logger.debug(cellClickEvent, "getVMRowIndex", $grid?.getVMRowIndex(row));
//     resolve(selectCurrent);
//   });
// };

/* 解决 tree 树形表格的单元格 edit 状态无法撑满 td 的问题 */
const cellClassName: VxeTablePropTypes.CellClassName = ({ row, column }) => {
  const cls = []
  const select = xGrid.value?.getSelectedCell()
  const active = xGrid.value?.getEditRecord()
  const current = xGrid.value?.getCurrentRecord()
  const isCell = (row === select?.row && column === select?.column) || (row === current?.row && column === current?.column)
  if (isCell) {
    logger.debug('cellClassName', {isCell,select,active,current})
  }
  if (isCell) {
    // 此处必须把 col--selected 返回，否则默认的单元格选中功能的边框将会失效
    cls.push(...['no-indent', 'col--selected'])
  } 
  if (column.type === 'checkbox') {
    // 返回 has-checkbox，解决只有一个 checkbox 时，checkbox 无法上下居中的问题
    cls.push('has-checkbox')
  }
  return cls.join(' ')
};

// 选中行边框设置
const rowClassName: VxeTablePropTypes.RowClassName = ({ row }) => {
  const select = xGrid.value?.getSelectedCell()
  const active = xGrid.value?.getEditRecord()
  const current = xGrid.value?.getCurrentRecord()
  
  const isRow = row === select?.row || row === current?.row || row === active?.row
  if (isRow) {
    logger.debug('rowClassName', {isRow,select,active,current})
  }
  if (isRow) {
    return "row-current";
  }
  return null;
};

// 行拖拽
let sortable: Sortable;
const rowDrop = () => {
  const $grid = xGrid.value;
  sortable = Sortable.create($grid?.$el.querySelector(".body--wrapper>.vxe-table--body tbody"), {
    handle: ".drag-btn",
    onEnd: (sortableEvent) => {
      const newIndex = sortableEvent.newIndex as number;
      const oldIndex = sortableEvent.oldIndex as number;
      const currRow = data.splice(oldIndex, 1)[0];
      data.splice(newIndex, 0, currRow);
    },
  });
};
let initTime: NodeJS.Timeout;
nextTick(() => {
  // 加载完成之后在绑定拖动事件
  initTime = setTimeout(() => {
    rowDrop();
  }, 500);
});

onUnmounted(() => {
  clearTimeout(initTime);
  if (sortable) {
    sortable.destroy();
  }
});

// 事件注册
const gridEvents: VxeGridListeners = {
  currentChange(params) {
    logger.info('currentChange')
  },
  cellAreaSelectionAllStart() {
    logger.info('cellAreaSelectionAllStart')
  },
  ActiveCellChangeStart() {
    logger.info('ActiveCellChangeStart')
  },
  editActived() {
    logger.info('editActived')
  },
  keydown(params) {
    const { row, column } = params
    logger.debug('keydown', {column,row})
  },
  headerCellClick (params) {
    const { column } = params
    console.log(`表头单元格点击${column.title}`)
  },
  headerCellDblclick (params) {
    const { column } = params
    console.log(`表头单元格双击${column.title}`)
  },
  headerCellMenu (params) {
    const { column } = params
    console.log(`表头右键单元格 ${column.title}`)
  },
  cellClick (params) {
    const { row, column } = params
    // const $xGrid = xGrid.value
    // logger.debug('cellClick', 'xGrid', $xGrid)
    // logger.debug('cellClick', {column,row})
    // logger.debug('status', status)
    // cellClickEvent(params)
  },
  cellDblclick (params) {
    const { column } = params
    console.log(`单元格双击${column.title}`)
  },
  cellMenu (params) {
    const { row } = params
    console.log(`单元格右键行 ${row.name}`)
  },
  footerCellClick (params) {
    const { column } = params
    console.log(`表尾单元格点击${column.title}`)
  },
  footerCellDblclick (params) {
    const { column } = params
    console.log(`表尾单元格双击${column.title}`)
  },
  footerCellMenu (params) {
    const { column } = params
    console.log(`表尾右键单元格 ${column.title}`)
  },
  checkboxChange (params) {
    console.log(`复选框切换 ${params.checked}`)
  },
  checkboxAll (params) {
    console.log(`复选框全选切换 ${params.checked}`)
  },
  scroll (params) {
    const bottomHeight = params.scrollHeight - params.scrollTop - params.bodyHeight
    // logger.info(gridEvents, 'scroll', bottomHeight)

    // 滚动到底部时，加载更多数据

  },
  zoom (params) {
    console.log(`表格全屏 type=${params.type}`)
  },
  custom (params) {
    console.log(`表格自定义列表 type=${params.type}`)
  }
}
</script>

<template>
  <div>
    <div class="h-38">
      <h1>Test</h1>
      <!-- <vxe-button @click="add">add</vxe-button>
    <vxe-button @click="remove">remove</vxe-button>
    <vxe-button @click="modify(10001)">modify</vxe-button> -->
      <vxe-button @click="clickEvent1">读取文件</vxe-button>
      <vxe-button @click="printEvent1">打印</vxe-button>
      <vxe-button @click="insertRowEnd(10)">在尾部插入空白行</vxe-button>
      <vxe-button @click="insertRowEndBySource(10)">在尾部插入数据By Source</vxe-button>
      <vxe-button @click="modifyCell(10050, 'name', Math.random())">修改数据</vxe-button>
      <vxe-button @click="removeRow(10050)">删除单行数据</vxe-button>
      <vxe-button @click="removeRow(10050, 24300, 20045, 10053, 24577, 24566, 24555)">删除多行数据</vxe-button>
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
    </div>
    <div class="fixed h-100% w-100%">
      <vxe-grid
        ref="xGrid"
        height="100%"
        :cell-class-name="cellClassName"
        :row-class-name="rowClassName"
        v-bind="gridOptions"
        v-on="gridEvents"
      >
        <template #text_show_change="{ row }">
          <!-- <span :tabindex="1" @change="cellChange">{{ row.name }}</span> -->
          <show-change-cell :value="row.name" :version="version"/>
        </template>
        <template #text_edit="{ row }">
          <!-- <vxe-input v-model="row.name"></vxe-input> -->
          <!-- 解决 vxe-inpu 无法 v-mode.lazy 的问题-->
          <input
            class="vxe-input--inner"
            :spellcheck="false"
            autocomplete="hidden"
            style="overflow: visible; resize: none; border-radius: 0; border: none; padding: 0; background: transparent"
            v-model.lazy="row.name"
          />
        </template>
        <template #ml_text_edit="{ row }">
          <!-- <vxe-input v-model="row.name"></vxe-input> -->
          <!-- 解决 vxe-input 无法 v-mode.lazy 的问题-->
          <textarea
            class="vxe-input--inner"
            style="overflow: visible; resize: none; border-radius: 0; border: none; padding: 0; background: transparent"
            :cols="5"
            :spellcheck="false"
            autocomplete="hidden"
            @keyup.enter.preventDefault()
            @keypress.enter.preventDefault()
            @keydown.enter.preventDefault()
            wrap="hard"
            v-model.lazy="row.name"
          ></textarea>
        </template>
        <template #text_edit2="{ row, column }">
          <!-- 实现数据流从数据源到表格的单项传递:修改单元格数据 -->
          <input
            class="vxe-input--inner"
            style="overflow: visible; resize: none; border-radius: 0; border: none; padding: 0; background: transparent"
            :spellcheck="false"
            autocomplete="hidden"
            :value="row[column.property]"
            @change="modifyCell(row.id, column.property, ($event.target as HTMLInputElement).value)"
          />
        </template>
        <template #action="{ row }">
          <!-- <vxe-button type="text" status="primary" @click="insertRow(row, 'current')">插入节点</vxe-button>
          <vxe-button type="text" status="primary" @click="insertRow(row, 'top')">顶部插入节点</vxe-button>
          <vxe-button type="text" status="primary" @click="insertRow(row, 'bottom')">尾部插入子节点</vxe-button>
          <vxe-button type="text" status="primary" @click="removeRow(row)">删除节点</vxe-button> -->
          <vxe-button type="text" status="primary" @click="insertRowBySource(row, 'before')">上一行插入</vxe-button>
          <vxe-button type="text" status="primary" @click="insertRowBySource(row, 'after')">下一行插入</vxe-button>
          <vxe-button type="text" status="primary" @click="insertRowBySource(row, 'last')">最后一行插入</vxe-button>
          <!-- <vxe-button type="text" status="primary" @click="insertRowBySource(row)">删除节点</vxe-button> -->
        </template>
        <template #index="{ $rowIndex }">
          <txt-cell class="drag-btn" :value="$rowIndex + 1" />
        </template>
        <template #text="{ $rowIndex }">
          <txt-cell :value="$rowIndex + 1" />
        </template>
        <template #checkbox="{ $rowIndex }">
          <txt-cell />
        </template>
      </vxe-grid>
    </div>
  </div>
</template>

<style>
/* 默认单元格设置 */
.vxe-table--render-default .vxe-cell {
  /* 解决 cell 的上下左右边距过宽的问题 */
  padding: 0 0;
  /* 解决 td 内元素无法撑满 td 的问题 */
  height: 100%;
}

/* 解决 cell 在编辑状态下，无选中边框的问题，改设置可以使其与选中状态 .col--selected 保持一致 */
.vxe-table--render-default .vxe-body--column.col--actived {
  box-shadow: inset 0 0 0 2px #409eff;
}

/* 解决只有一个 checkbox 时，checkbox 无法上下居中的问题 */
.vxe-table--render-default .vxe-body--column.has-checkbox .vxe-cell {
  height: auto;
}

.vxe-table--render-default .vxe-cell .vxe-cell--tree-node {
  /* 解决 td 内元素无法撑满 td 的问题 */
  height: 100%;
}

.vxe-table--render-default .vxe-cell .vxe-cell--tree-node .vxe-tree-cell {
  /* 解决 td 内元素无法撑满 td 的问题 */
  height: 100%;
}

/* 解决修改数据/新增行的单元格左上角“三角”标识 */
.vxe-table .vxe-body--column::before {
  display: none;
}

/* 解决 tree 树形表格的单元格 edit 状态无法撑满 td 的问题 */
.vxe-table .vxe-body--column.no-indent .vxe-cell--tree-node {
  padding-left: 0 !important;
}
.vxe-table .vxe-body--column.no-indent .vxe-tree-cell {
  padding-left: 0 !important;
}
.vxe-table .vxe-body--column.no-indent .vxe-tree--btn-wrapper {
  display: none;
}

/* 选中行边框显示 */
.vxe-table .vxe-body--row.row-current {
  outline: 1px solid rgba(0, 0, 0, 0.5);
  outline-offset: -3px;
}

/* 修改默认的 row-config.isCurrent 样式，使其与 选中行边框一致 */
/* .vxe-table .vxe-body--row.row--current {
  outline: 1px solid rgba(0, 0, 0, 0.5);
  outline-offset: -3px;
} */

/* 拖拽鼠标样式 */
.vxe-table .drag-btn {
  cursor: move;
  font-size: 12px;
}
.vxe-table .vxe-body--row.sortable-ghost,
.vxe-table .vxe-body--row.sortable-chosen {
  background-color: #dfecfb;
}
</style>
