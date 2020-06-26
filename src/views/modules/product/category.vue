<template>
  <div>
    <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽" style="marginRight: 15px"></el-switch>
    <el-button v-if="draggable" type="primary" @click="batchSave" style="marginBottom: 10px">批量保存</el-button>
    <el-button type="danger" @click="batchDelete" style="marginBottom: 10px">批量删除</el-button>

    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >Append</el-button>
          <el-button
            type="text"
            size="mini"
            @click="edit(data)"
          >Edit</el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>

    <!-- 添加/修改 对话框 -->
    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
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
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name"
      },
      dialogVisible: false,
      expandedKey: [],   //默认展开的节点
      dialogType: "",    //"edit" OR "add"
      dialogTitle: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
        catId: null
      },
      maxLevel: 0,
      updateNodes: [],    //存放 被拖拽节点、被拖拽节点的兄弟节点、被拖拽节点的子节点
      parentCatId: [],    //被拖拽节点 最新的父节点id
      draggable: false,   //是否开启拖拽功能
    };
  },
  computed: {},
  watch: {},
  methods: {
    // 批量删除
    batchDelete() {  
      // 获取目前被选中的节点
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      
      let deleteCatIds = [];
      for (let i = 0; i < checkedNodes.length; i++) {
        deleteCatIds.push(checkedNodes[i].catId);
      }
      
      this.$confirm(`是否批量删除【${deleteCatIds}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(deleteCatIds, false)
        }).then(({ data }) => {
          this.$message({
            message: "菜单批量删除成功",
            type: "success"
          });
          this.getMenus();
        });
      }).catch(() => {

      });
    },

    // 批量保存
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/batch/update"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          message: "修改成功",
          type: "success"
        });
        // 刷新菜单
        this.getMenus();
        // 设置 默认展开的节点 - 默认展开所有被拖拽节点的父节点
        this.expandedKey = this.parentCatId;
        // 初始化 updateNodes 和 maxLevel
        this.updateNodes = [];
        this.maxLevel = 0;
      });
    },
    // 拖拽成功完成时触发的事件
    // 共四个参数，依次为：被拖拽节点对应的 Node、结束拖拽时最后进入的节点、被拖拽节点的放置位置（before、after、inner）、event
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 目标
      //   1.得到 被拖拽节点 最新的父节点id、最新的排序、最新的层级
      //   2.将 被拖拽节点、被拖拽节点的兄弟节点、被拖拽节点的子节点 存入 updateNodes数组 中

      // 最新的父节点id
      let parentCatId = 0;
      // 最新的兄弟节点 (siblings中包含被拖拽节点)
      let siblings = null;
      
      // 得到 被拖拽节点 最新的父节点id 和 最新的兄弟节点
      if (dropType == "before" || dropType == "after") {
        parentCatId = dropNode.parent.data.catId == undefined ? 0 : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        parentCatId = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.parentCatId.push(parentCatId);

      // 遍历siblings，生成每个兄弟节点的最新排序值，同时得到 被拖拽节点 最新的层级
      for (let i = 0; i < siblings.length; i++) {
        // 判断 当前遍历节点 是否为 被拖拽节点
        if (siblings[i].data.catId == draggingNode.data.catId) {
          let catLevel = draggingNode.level;
          // 判断 被拖拽节点 拖拽前后 层级是否发生变化，如果发生了变化，那就需要修改 被拖拽节点的所有子节点 的 层级
          if (siblings[i].level != catLevel) {
            // 拖拽前后 层级发生了变化 : 得到最新层级，然后递归修改 被拖拽节点的所有子节点 的 层级
            catLevel = siblings[i].level;
            this.updateChildNodeLevel(siblings[i]);
          }
          // 将 被拖拽节点 存入数组
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: parentCatId,
            catLevel: catLevel
          });
        } else {
          // 将 被拖拽节点的兄弟节点 存入数组
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
    },
    // 递归修改 被拖拽节点的所有子节点 的 层级
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          // 将 被拖拽节点的子节点 存入数组
          this.updateNodes.push({
            catId: node.childNodes[i].data.catId,
            catLevel: node.childNodes[i].level
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    // 拖拽时判定目标节点能否被放置
    //   draggingNode：当前正在被拖拽的节点
    //   dropNode：目标节点 (draggingNode 被拖拽到 dropNode 处)
    //   type 参数有三种情况：'prev'、'inner' 和 'next'，分别表示放置在目标节点前、插入至目标节点和放置在目标节点后
    allowDrop(draggingNode, dropNode, type) {
      // 判定目标节点能否被放置 => 判断 目标节点的level + 以被拖拽节点为根的子树的总层数 是否大于 3 : 如果 > 3，则不能放置 ; 如果 <= 3，则可以放置

      // 得出 被拖拽节点 (包括它的子节点) 最大深度
      // e.g. 被拖拽节点位于整个菜单树的第2层，它的最深子节点位于整个菜单树的第5层 -> 最大深度为5，因为被拖拽节点的最深子节点位于整个菜单树的第5层
      this.maxLevel = draggingNode.level;
      this.countNodeLevel(draggingNode);
      // 得出 被拖拽节点 (包括它的子节点) 一共有多少层
      // e.g. 被拖拽节点位于整个菜单树的第2层，它的最深子节点位于整个菜单树的第5层 -> 一共有4层，因为从第2层到第5层一共有4层
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;

      if (type === "inner") {
        // 拖拽到目标节点内部
        return deep + dropNode.level <= 3;
      } else {
        // 拖拽到目标节点前 OR 拖拽到目标节点后
        return deep + dropNode.parent.level <= 3;
      }
    },
    // 递归得出 被拖拽节点的子节点 最大深度
    countNodeLevel(node) {
      if (node.childNodes !== null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },

    submitData() {
      if (this.dialogType === "add") {
        this.addCategory();
      }
      if (this.dialogType === "edit") {
        this.editCategory();
      }
    },

    // 修改分类
    editCategory() {
      let { catId, name, icon, productUnit } = this.category;

      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功",
          type: "success"
        });
        // 关闭对话框
        this.dialogVisible = false;
        // 刷新菜单
        this.getMenus();
        // 设置 默认展开的节点 - 默认展开被修改节点的父节点
        this.expandedKey = [this.category.parentCid];
      });
    },
    // 打开 修改 对话框
    edit(data) {
      this.dialogVisible = true;
      this.dialogType = "edit";
      this.dialogTitle = "修改分类";

      // 根据分类id获取分类信息 - 用于回显分类信息
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
        this.category.catLevel = data.data.catLevel;
        this.category.sort = data.data.sort;
        this.category.showStatus = data.data.showStatus;
      });
    },

    // 添加分类
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功",
          type: "success"
        });
        // 关闭对话框
        this.dialogVisible = false;
        // 刷新菜单
        this.getMenus();
        // 设置 默认展开的节点 - 默认展开被添加节点的父节点
        this.expandedKey = [this.category.parentCid];
      });
    },
    // 打开 添加 对话框
    append(data) {
      this.dialogVisible = true;
      this.dialogType = "add";
      this.dialogTitle = "添加分类";

      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },

    // 删除单个节点
    remove(node, data) {
      let ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            message: "菜单删除成功",
            type: "success"
          });
          // 刷新菜单
          this.getMenus();
          // 设置 默认展开的节点 - 默认展开被删除节点的父节点
          this.expandedKey = [node.parent.data.catId];
        });
      }).catch(() => {

      });
    },

    getMenus() {
      // 查询所有商品分类信息
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        this.menus = data.data;
      });
    }
  },
  created() {
    this.getMenus();
  },
  mounted() {},
  beforeCreate() {},
  beforeMount() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  activated() {}
};
</script>

<style scoped>
</style>