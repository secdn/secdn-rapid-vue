<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="栏目名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="请输入栏目名称"></el-input>
      </el-form-item>
      <el-form-item label="上级栏目" prop="name">
        <el-popover ref="menuListPopover" placement="bottom-start" trigger="click">
          <el-tree
            :data="menuList"
            :props="menuListTreeProps"
            node-key="id"
            ref="menuListTree"
            @current-change="menuListTreeCurrentChangeHandle"
            :default-expand-all="true"
            :highlight-current="true"
            :expand-on-click-node="false"
          ></el-tree>
        </el-popover>
        <el-input
          v-model="dataForm.parentName"
          v-popover:menuListPopover
          :readonly="true"
          placeholder="点击选择上级菜单"
          class="menu-list__input"
        ></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="description">
        <el-input v-model="dataForm.description" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="排序号" prop="sort">
        <el-input-number v-model="dataForm.sort" controls-position="right" :min="0" label="排序号"></el-input-number>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { treeDataTranslate } from "@/utils";
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        name: "",
        pid: 0,
        parentName: "",
        sort: 0,
        description: ""
      },
      dataRule: {
        name: [{ required: true, message: "菜单名称不能为空", trigger: "blur" }]
      },
      menuList: [],
      menuListTreeProps: {
        label: "name",
        children: "children"
      }
    };
  },
  created() {},
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.$http({
        url: this.$http.adornUrl("/bizType/getBizTypeList"),
        method: "post",
        data: this.$http.adornData()
      })
        .then(({ data }) => {
          let list = data.result.records;
          this.menuList = treeDataTranslate(list,"id","pid");
        })
        .then(() => {
          this.visible = true;
          this.$nextTick(() => {
            this.$refs["dataForm"].resetFields();
          });
        })
        .then(() => {
          if (!this.dataForm.id) {
            // 新增
            this.menuListTreeSetCurrentNode();
          } else {
            // 修改
            this.$http({
              url: this.$http.adornUrl(`/bizType/getBizTypeById/${this.dataForm.id}`),
              method: "post",
              params: this.$http.adornData()
            }).then(({ data }) => {
              this.dataForm.id = data.result.bizType.id;
              this.dataForm.name = data.result.bizType.name;
              this.dataForm.pid = data.result.bizType.pid; 
              this.dataForm.sort = data.result.bizType.sort;
              this.dataForm.parentName = data.result.bizType.parentName;
              this.dataForm.description = data.result.bizType.description;
              this.menuListTreeSetCurrentNode();
            });
          }
        });
    },
    // 菜单树选中
    menuListTreeCurrentChangeHandle(data, node) {
      this.dataForm.pid = data.id;
      this.dataForm.parentName = data.name;
    },
    // 菜单树设置当前选中节点
    menuListTreeSetCurrentNode() {
      this.$refs.menuListTree.setCurrentKey(this.dataForm.pid);
      this.dataForm.parentName = (this.$refs.menuListTree.getCurrentNode() ||
        {})["name"];
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/bizType/${!this.dataForm.id ? "addBizType" : "updateBizTypeById"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              parentName: this.dataForm.parentName,
              pid: this.dataForm.pid,
              sort: this.dataForm.sort,
              description: this.dataForm.description
            })
          }).then(({ data }) => {
            if (data && data.code === 200) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.message);
            }
          });
        }
      });
    }
  }
};
</script>

<style lang="scss">
.mod-menu {
  .menu-list__input,
  .icon-list__input {
    > .el-input__inner {
      cursor: pointer;
    }
  }
  &__icon-popover {
    width: 458px;
    overflow: hidden;
  }
  &__icon-inner {
    width: 478px;
    max-height: 258px;
    overflow-x: hidden;
    overflow-y: auto;
  }
  &__icon-list {
    width: 458px;
    padding: 0;
    margin: -8px 0 0 -8px;
    > .el-button {
      padding: 8px;
      margin: 8px 0 0 8px;
      > span {
        display: inline-block;
        vertical-align: middle;
        width: 18px;
        height: 18px;
        font-size: 18px;
      }
    }
  }
  .icon-list__tips {
    font-size: 18px;
    text-align: center;
    color: #e6a23c;
    cursor: pointer;
  }
}
</style>
