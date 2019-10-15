<template>
  <div class="add_or_update">
    <el-breadcrumb separator="/" style="padding: 20px 10px;">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/app-apparticle' }">文章列表</el-breadcrumb-item>
      <el-breadcrumb-item v-if="id == undefined">新增文章</el-breadcrumb-item>
      <el-breadcrumb-item v-else>活动详情</el-breadcrumb-item>
    </el-breadcrumb>
    <el-row>
      <el-form ref="dataForm" :model="dataForm" label-width="80px" :rules="dataRule">
        <el-form-item label="标题：" prop="title">
          <el-col :span="12">
            <el-row>
              <el-col :span="10">
                <el-input v-model="dataForm.title"></el-input>
              </el-col>
              <el-col :span="2">
                <span v-show="contentShortLength" class="word-counter">{{ contentShortLength }}字</span>
              </el-col>
            </el-row>
          </el-col>
        </el-form-item>
        <el-form-item label="栏目：" prop="typeName">
          <el-col :span="4">
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
              v-model="dataForm.typeName"
              v-popover:menuListPopover
              :readonly="true"
              placeholder="点击选择栏目"
              class="menu-list__input"
            ></el-input>
          </el-col>
        </el-form-item>
        <el-form-item label="内容：" prop="content" style="margin-bottom: 30px;">
          <tinymce ref="editor" :height="400" v-model="dataForm.content"/>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">{{btnTitle}}</el-button>
          <el-button @click="onCancle">取消</el-button>
        </el-form-item>
      </el-form>
    </el-row>
  </div>
</template>
<script>
import { treeDataTranslate, formatTime } from "@/utils";
import Tinymce from "@/components/Tinymce";
export default {
  components: { Tinymce },
  data() {
    return {
      id: this.$route.query.id,
      dataForm: {
        id: 0,
        title: "",
        subTitle: "",
        createTime: formatTime(Date.now() / 1000, "Y-M-D h:m:s"),
        content: "",
        typeId: "",
        typeName: "",
        userId: ""
      },
      dataRule: {
        title: [{ required: true, message: "标题不能为空", trigger: "blur" }],
        typeName: [
          { required: true, message: "栏目名称不能为空", trigger: "blur" }
        ],
        content: [
          { required: true, message: "文章内容不能为空", trigger: "blur" }
        ]
      },
      menuList: [],
      menuListTreeProps: {
        label: "name",
        children: "children"
      },
      btnTitle:this.$route.query.id?"更新文章":"创建文章"
    };
  },
  computed: {
    contentShortLength() {
      return this.dataForm.title.length;
    }
  },
  mounted() {
    this.handleApptypeList();
  },
  methods: {
    handleApptypeList() {
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
          console.log(this.$route.query.id)
          if (this.$route.query.id) {
            // 修改
            this.$http({
              url: this.$http.adornUrl(`/bizArticle/getBizArticleById/${this.$route.query.id}`),
              method: "post",
              params: this.$http.adornData()
            }).then(({ data }) => {
              console.log(data)
              this.dataForm.id = data.result.bizArticle.id;
              this.dataForm.title = data.result.bizArticle.title;
              this.dataForm.createTime = data.result.bizArticle.createTime;
              this.dataForm.sort = data.result.bizArticle.sort;
              this.dataForm.typeName = data.result.bizArticle.typeName;
              this.dataForm.typeId = data.result.bizArticle.typeId;
              this.dataForm.content = data.result.bizArticle.content;
              this.menuListTreeSetCurrentNode();
            });
          }
        });
    },
    // 菜单树选中
    menuListTreeCurrentChangeHandle(data, node) {
      this.dataForm.typeId = data.id;
      this.dataForm.typeName = data.name;
    },
    // 菜单树设置当前选中节点
    menuListTreeSetCurrentNode() {
      this.$refs.menuListTree.setCurrentKey(this.dataForm.typeId);
      this.dataForm.typeName = (this.$refs.menuListTree.getCurrentNode() || {})[
        "name"
      ];
    },
    onSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/bizArticle/${!this.dataForm.id ? "addBizArticle" : "updateBizArticleById"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              title: this.dataForm.title,
              typeName: this.dataForm.typeName,
              typeId: this.dataForm.typeId,
              content: this.dataForm.content,
              createTime: this.dataForm.createTime,
              subTitle: this.dataForm.subTitle
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
                  this.onCancle()
                }
              });
            } else {
              this.$message.error(data.message);
            }
          });
        }
      });
    },
    onCancle(){
      this.$router.back(-1)
    }
  }
};
</script>
<style lang="scss" scoped>
.word-counter{
  margin-left: 20px;
}
</style>
