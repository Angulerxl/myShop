<template>
  <div>
    <!-- 内容 -->
    <!-- <el-checkbox v-model="checkAll" @change="handleCheckAll">全选</el-checkbox>
    <div style="margin-bottom: 10px;"></div>
    <el-checkbox-group v-model="checkedIds" @change="handleChange">
      <el-checkbox v-for="item in downBtns" :label="item.value" :key="item.value">{{item.label}}</el-checkbox>
    </el-checkbox-group> -->

    <div class="main-btn">
      第三步：<el-button class="main-btn1" type="success" @click="onOk">导出excel</el-button>
    </div>
    <el-divider></el-divider>
    <!-- {{_EXPORT_DATAS.doOrdersDataSource['_sameBuyerDataSource']}} -->
    <!-- ---------- -->
    第四步：
    <span slot="footer" class="dialog-footer">
      <el-button class="order-sn" size="small" type="info" @click="exportSn">
        <i class="el-icon-download"></i>单号汇总</el-button>
      <download-excel :ref="`download-img`" :class="['export-img img']" :data="imgExport.data || []" :name="imgExport.name">
        <i class="el-icon-download"></i>下载img
      </download-excel>
      <!-- <download-excel :ref="`download`" :class="['export-img doOrder']" :data="orderExport.data || []" :name="orderExport.name">
          {{item.label}}
        </download-excel> -->
      <span v-for="item in downBtns" :key="item.value">
        <download-excel :ref="`download`" :class="['export-img doOrder']" :data="orderExportObj[item.valKey] || []"
                        :name="`${fileName}做单--${item.label}-${timer}.xlsx`">
          <i class="el-icon-download"></i>
          {{item.label}}
          <span v-if="_EXPORT_DATAS.doOrdersDataSource">
            （{{_EXPORT_DATAS.doOrdersDataSource[item.valKey].length>0?'有':'无'}}）
          </span>
        </download-excel>
      </span>
    </span>

    <!-- 不常用 -->

    <div class="zdy-ipt">

      下载文件名: <el-input size="small" style="width: 240px;margin-right:16px" v-model="fileName" placeholder="下载文件名"></el-input>
      excel的索引： <el-input size="small" style="width: 60px" v-model="startIndex" placeholder="开始索引,默认1"></el-input>
    </div>
  </div>
</template>

<script>
import {
  filter as _filter,
  cloneDeep as _cloneDeep,
  includes as _includes,
  find as _find,
  findIndex as _findIndex,
  uniqBy as _uniqBy,
  get as _get,
} from "lodash";
import { saveAs } from "file-saver";
import { _snTxtStr, getDay } from "./utils/main.js";
import { _exportTabel as _exportImg } from "./utils/imgExport.js";
import { _orderTabel as _exportOrder } from "./utils/ordersExport.js";
const __win_data = JSON.parse(window.localStorage.getItem("__sys2-base"));

//默认选中
const options = _cloneDeep(__win_data?.btns);
const initIds = options?.map((item) => item.value);
const d = _cloneDeep(initIds);
const checkedIds = d;

export default {
  props: {
    _EXPORT_DATAS: {
      type: Object,
      default: () => {},
    },
  },
  data() {
    return {
      downBtns: __win_data.btns,
      checkAll: false,
      checkedIds: checkedIds,
      fileName: "",
      timer: "",
      startIndex: "1",
      imgExport: {
        //报货
        data: [],
        name: "",
      },
      orderExport: {
        data: [],
        name: "",
      },
      orderExportObj: {},
    };
  },
  computed: {},
  methods: {
    onOk() {
      if (!this._EXPORT_DATAS.allOrderCodes) {
        this.$message.error("请先导入数据");
        return;
      }
      this.timer = getDay();
      this.exportImg(this._EXPORT_DATAS.flatDataSource);
      this.exportOrders(this._EXPORT_DATAS.doOrdersDataSource);
      this.$message.success("导出成功，快去下载");
    },
    //-----------------导出做单-----------------
    exportOrders() {
      this.orderExportObj = _exportOrder(
        this._EXPORT_DATAS.doOrdersDataSource,
        this.startIndex
      );
    },
    // exportOrders() {
    //   const resObj = _exportOrder(
    //     this._EXPORT_DATAS.doOrdersDataSource,
    //     this.startIndex
    //   );
    //   this.orderExport.data = resObj._customizedDataSource;

    //   this.orderExport.name = `${this.fileName || ""}做单-${this.timer}.xlsx`;
    // },
    //-----------------导出报货-----------------
    exportImg() {
      this.imgExport.data = _exportImg(this._EXPORT_DATAS.flatDataSource); //图片汇总导出
      this.imgExport.name = `${this.fileName || ""}报货-${this.timer}.xlsx`;
    },
    //-----------------导出单号-----------------
    exportSn() {
      if (!this._EXPORT_DATAS.allOrderCodes) return;
      const codes4 = _snTxtStr(this._EXPORT_DATAS.allOrderCodes);
      const res = new Blob([codes4], {
        type: "text/plain;charset=utf-8",
      });
      const name = `${this.fileName || ""}单号-${this.timer}.txt`;
      saveAs(res, `${name}`);
    },
    // 多选框
    handleCheckAll(val) {
      this.checkedIds = val ? initIds : [];
      this.isIndeterminate = false;
    },
    handleChange(value) {
      let checkedCount = value.length;
      this.checkAll = checkedCount === this.datas.length;
      this.isIndeterminate =
        checkedCount > 0 && checkedCount < this.datas.length;
    },
    empty() {
      this.orderExportObj = {};
      this.orderExport.data = [];
      this.orderExport.name = "";
      this.imgExport.data = [];
      this.imgExport.name = "";
    },
  },
};
</script>
<style lang="less" scoped>
.export-img {
  // width: 60px;
  border-radius: 4px;
  font-size: 14px;
  padding: 2px 8px;
  cursor: pointer;
  color: #000;
  height: 30px;
  line-height: 30px;
  text-align: center;
  margin-left: 20px;
  // border: 1px solid #67c23a;
}
.img {
  background-color: #e6a23c;
}
.doOrder {
  background-color: #409eff;
}
:deep(.el-dialog__body) {
  text-align: left;
}
:deep(.el-checkbox) {
  display: block;
  margin-bottom: 8px;
}
:deep(.el-dialog__body) {
  padding: 0 20px;
}
.dialog-footer {
  display: flex;
  justify-content: center;
}
.main-btn {
  margin-left: 20px;
  margin-top: 16px;
  .main-btn1 {
    width: 100%;
    margin-bottom: 8px;
  }
}
.order-sn {
  color: #000;
}

:deep(.el-input__inner) {
  border: 1px solid #409eff;
}
.zdy-ipt {
  display: flex;
  margin-bottom: 24px;
  margin-top: 80px;
}
</style>
