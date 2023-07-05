<template>
  <div>
    <el-tree :data="menus" :props="defaultProps"
             :expand-on-click-node="false" node-key="catId"
             :show-checkbox="true">
        <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level<= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
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
      console.log("append", data)
    },

    remove(node, data) {
      let ids = [data.catId]
      this.$http({
        url: this.$http.adornUrl('/product/category/delete'),
        method: 'post',
        data: this.$http.adornData(ids, false)
      }).then(({data}) => {
        console.log("delete success")
        this.getMenus();
      })
    },
  },
  created() {
    this.getMenus();
  }
};
</script>
<style scoped>

</style>
