<template>
  <div>
    <el-dialog :title="$t('dialog.moveto-title')" width="30%" :visible.sync="visible">
      <el-tree :data="data" :props="props" node-key="id" :current-node-key="current.id" :default-expanded-keys="[0]" :load="loadNode" :highlight-current="true" @current-change="onCurrentChange" lazy>
        <span class="custom-tree-node" slot-scope="{ node }">
          <span>
            <i class="el-icon-folder"></i>
            {{ node.label }}
          </span>
        </span>
      </el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submit">{{ $t("confirm") }}</el-button>
        <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { CSMixin, DialogMixin } from "@/libs/mixin";
export default {
  mixins: [CSMixin, DialogMixin],
  props: {
    alias: String,
    isDir: Boolean,
  },
  data() {
    return {
      data: [],
      props: {
        label: "name",
        children: "folders",
        isLeaf: "leaf",
      },
      current: {},
      treectx: {},
    };
  },
  methods: {
    onCurrentChange(item, node) {
      this.current = item;
    },
    loadNode(node, resolve) {
      if (node.level === 0) {
        resolve([{ id: 0, name: "/", parent: "" }]);
        return;
      }

      // cache the args to refresh data
      if (!this.treectx.node && node.level === 1) {
        this.treectx.node = node;
        this.treectx.resolve = resolve;
      }

      // pull the datas
      this.$zpan.File.list({ sid: this.getSid(), dir: node.data.fullpath }).then((ret) => {
        let folders = ret.list.filter((ele) => {
          return ele.dirtype && ele.alias != this.alias;
        });
        resolve(folders);
      });
    },
    submit() {
      this.$zpan.File.move(this.alias, this.current.fullpath).then((ret) => {
        this.$message({
          type: "success",
          message: this.$t("msg.move-success"),
        });
        this.finish();
        this.close();
      });
    },
  },
};
</script>

<style scoped>
.custom-tree-node {
  font-size: 22px;
  line-height: 25px;
}
.el-icon-folder {
  color: #ffc402;
}
</style>
 No newline at end of file
