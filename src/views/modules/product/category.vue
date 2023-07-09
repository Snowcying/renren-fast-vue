<template>
  <div>
    <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-tree :allow-drop="allowDrop" :data="menus"
             :default-expanded-keys="expandedKey" :draggable="draggable"
             :expand-on-click-node="false"
             :props="defaultProps"
             :show-checkbox="true"
             node-key="catId"
             @node-drop="handleDrop">
        <span slot-scope="{ node, data }" class="custom-tree-node">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level<= 2"
            size="mini"
            type="text"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            size="mini"
            type="text"
            @click="() => remove(node, data)">
            Delete
          </el-button>
                    <el-button
                      size="mini"
                      type="text"
                      @click="() => edit(data)">
            edit
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogFormVisible"
      width="30%">

      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </div>

    </el-dialog>
  </div>
</template>

<script>
import fa from "element-ui/src/locale/lang/fa";

export default {
  computed: {
    fa() {
      return fa
    }
  },
  data() {
    return {
      data: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      menus: [],
      expandedKey: [],
      formLabelWidth: '120px',
      dialogFormVisible: false,
      category: {name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null, productUnit: "", icon: "",},
      dialogType: "",
      title: "",
      maxLevel: 0,
      updateNodes: [],
      pCid: [],
      draggable: false,
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
        // params: this.$http.adornParams({
        //   'page': this.pageIndex,
        //   'limit': this.pageSize,
        //   'roleName': this.dataForm.roleName
        // })
      }).then(({data}) => {
        console.log("success---!!", data.data)
        this.menus = data.data;
      })
    },
    append(data) {
      this.dialogType = "add";
      this.title = "添加分类"
      this.dialogFormVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1;

      this.category.name = "";
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
    },

    remove(node, data) {
      let ids = [data.catId]
      this.$http({
        url: this.$http.adornUrl('/product/category/delete'),
        method: 'post',
        data: this.$http.adornData(ids, false)
      }).then(({data}) => {
        console.log("delete success")
        this.$message({
          message: '菜单删除成功',
          type: 'success'
        });
        this.getMenus();
        this.expandedKey = [node.parent.data.catId]
      })
    },
    edit(data) {
      console.log(data);
      this.dialogType = "edit"
      this.title = "修改分类"
      this.dialogFormVisible = true;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({data}) => {
        console.log("回显数据", data)
        this.category.name = data.category.name;
        this.category.catId = data.category.catId;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid;
      })

    },
    addCategory() {
      console.log(this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单保存成功',
          type: 'success'
        });
        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      })
    },
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    editCategory() {
      let {catId, name, icon, productUnit} = this.category;
      // let data={catId: catId,name:name,icon: icon,productUnit: productUnit};

      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit}, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单修改成功',
          type: 'success'
        });
        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      })
    },
    allowDrop(draggingNode, dropNode, type) {
      //1、被拖动的当前节点以及所在的父节点总层数不能大于3

      //1）、被拖动的当前节点总层数
      console.log("allowDrop:", draggingNode, dropNode, type);
      //
      this.countNodeLevel(draggingNode.data);
      //当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = this.maxLevel - draggingNode.data.catLevel + 1;
      console.log("深度：", deep);

      //   this.maxLevel
      if (type == "inner") {
        // console.log(
        //   `this.maxLevel：${this.maxLevel}；draggingNode.data.catLevel：${draggingNode.data.catLevel}；dropNode.level：${dropNode.level}`
        // );
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      //找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].catLevel > this.maxLevel) {
            this.maxLevel = node.childNodes[i].catLevel;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log('handleDrop', draggingNode, dropNode, dropType);
      let pCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);
      //2、当前拖拽节点的最新顺序，
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          //如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            //当前节点的层级发生变化
            catLevel = siblings[i].level;
            //修改他子节点的层级
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel
          });
        } else {
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i});
        }
      }
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单顺序修改成功',
          type: 'success'
        });
        this.getMenus();
        this.expandedKey = [pCid];
        this.updateNodes = [];
        this.maxLevel = 0;
      })
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单顺序等修改成功",
          type: "success"
        });
        //刷新出新的菜单
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        // this.pCid = 0;
      });
    },
  },
  created() {
    this.getMenus();
  },
};
</script>
<style scoped>

</style>
