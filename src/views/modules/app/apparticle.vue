<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.title" placeholder="角色名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('app:apptype:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button
          v-if="isAuth('app:apparticle:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >批量删除</el-button>
      </el-form-item>

       <el-select v-model="typeVal" filterable placeholder="请选择栏目" @change="handleChange">
      <el-option
        v-for="item in typeList"
        :key="item.id"
        :label="item.name"
        :value="item.id"
        >
      </el-option>
  </el-select>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
      :default-sort = "{prop: 'id', 'title': 'descending'}"
    >
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" header-align="center" align="center" width="120" label="ID" sortable></el-table-column>
      <el-table-column prop="title" header-align="center" align="center" label="文章标题" sortable></el-table-column>
      <el-table-column prop="typeName" header-align="center" align="center" label="栏目名称" sortable></el-table-column>
      <el-table-column prop="remark" header-align="center" align="center" label="备注"></el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        width="180"
        label="创建时间"
        sortable
      ></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button
            v-if="isAuth('app:apparticle:update')"
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
          >修改</el-button>
          <el-button
            v-if="isAuth('app:apparticle:delete')"
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
  </div>
</template>

<script>
export default {
  data() {
    return {
      dataForm: {
        title: ""
      },
      dataList: [],
      pageNumber: 1,
      pageSize: 10,
      totalPage: 0,
      typeId:"",
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      typeList: [],
      typeVal: ''
    };
  },
  components: {
    //   AddOrUpdate
  },
  activated() {
    this.getDataList();
    this.getTypeList();
  },
  methods: {
    handleChange(e){
      this.typeId = e;
      this.getDataList()
    },
    getTypeList() {
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
          let newOpt = {
            name:"全部",
            id:''
          }
          list.unshift(newOpt)
          this.typeList = list;
        } else {
          this.typeList = [];
        }
      });
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/bizArticle/queryAllList"),
        method: "post",
        data: this.$http.adornData({
          pageSize:String(this.pageSize),
          pageNumber:String(this.pageNumber),
          typeId:String(this.typeId)
        })
      }).then(({ data }) => {
        if (data && data.code === 200) {
          this.dataList = data.result.records;
          this.totalPage = data.result.totalCount;
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
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 修改
    addOrUpdateHandle(id) {
      this.$nextTick(() => {
        this.$router.push({
          path: '/add_article',
          query: {
            id: id
          }
        });
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/bizArticle/delete"),
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
