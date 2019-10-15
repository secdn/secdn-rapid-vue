<template>
  <div class="mod-menu">
    <el-form :inline="true" :model="dataForm">
      <el-form-item>
        <el-button v-if="isAuth('app:apptype:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border style="width: 100%;" :default-sort = "{prop: 'name', sort: 'descending'}">
      <el-table-column prop="id" header-align="center" align="center" width="80" label="ID"></el-table-column>
      <table-tree-column
        prop="name"
        header-align="center"
        treeKey="id"
        width="350"
        label="栏目名称"
        parentKey="pid"
        sortable
      ></table-tree-column>
      <el-table-column prop="parentName" header-align="center" align="center" label="上级栏目"></el-table-column>
      <el-table-column prop="description" header-align="center" align="center" label="描述"></el-table-column>
      <el-table-column prop="sort" header-align="center" align="center" label="排序号" sortable></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button
            v-if="isAuth('app:apptype:update')"
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
          >修改</el-button>
          <el-button
            v-if="isAuth('app:apptype:delete')"
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageNumber"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import TableTreeColumn from "@/components/table-tree-column";
import AddOrUpdate from "./type-add-or-update";
import { treeDataTranslate } from "@/utils";
export default {
  data() {
    return {
      dataForm: {},
      dataList: [],
      dataListLoading: false,
      addOrUpdateVisible: false,
      pageNumber:1,
      pageSize:10,
      totalPage: 0
    };
  },
  components: {
    TableTreeColumn,
    AddOrUpdate
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/bizType/getBizTypeList"),
        method: "post",
        data: this.$http.adornData({
          pageSize:String(this.pageSize),
          pageNumber:String(this.pageNumber)
        })
      }).then(({ data }) => {
        if (data && data.code === 200) {
          let list = data.result.records;
          this.totalPage = data.result.totalCount
          this.dataList = treeDataTranslate(list, "id", "pid");
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageNumber = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageNumber = val;
      this.getDataList();
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      this.$confirm(`确定对[id=${id}]进行[删除]操作?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          var ids = id
            ? [id]
            : this.dataListSelections.map(item => {
                return item.id;
              });
          this.$http({
            url: this.$http.adornUrl(`/bizType/delete`),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            if (data && data.code === 200) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.getDataList();
                }
              });
            } else {
              this.$message.error(data.message);
            }
          });
        })
        .catch(() => {});
    }
  }
};
</script>
