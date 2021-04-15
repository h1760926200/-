<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoulesVisiable = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>

      <el-table :data="rolelist" border stripe>
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row
              :class="['bd-bottom', 'vcenter', index1 == 0 ? 'bd-top' : '']"
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[index2 == 0 ? '' : 'bd-top', 'vcenter']"
                  v-for="(item2, index2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      type="success"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="450px">
          <template v-slot="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              @click="editRoulesForm(scope.row.id)"
              size="mini"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="DeleteRoules(scope.row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 编辑角色对话框 -->
    <el-dialog
      title="编辑角色"
      :visible.sync="editRoulesVisiable"
      width="30%"
      @close="editrouleClosed"
    >
      <el-form
        ref="editRoulesRef"
        :model="editroulesForm"
        :rules="editroulesRules"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editroulesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editroulesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoulesVisiable = false">取 消</el-button>
        <el-button type="primary" @click="editRoules">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoulesVisiable"
      width="30%"
      @close="addrouleClosed"
    >
      <el-form ref="addRoulesRef" :model="roulesForm" :rules="roulesRules">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roulesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="roulesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoulesVisiable = false">取 消</el-button>
        <el-button type="primary" @click="addRoules">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      @close="setRightDialogClosed"
      width="50%"
    >
      <el-tree
        :data="rightslist"
        :default-checked-keys="defKeys"
        node-key="id"
        ref="treeRef"
        default-expand-all
        show-checkbox
        :props="treeProps"
      >
      </el-tree>
      <div slot="footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolelist: [],
      setRightDialogVisible: false,
      rightslist: [],
      treeProps: {
        children: "children",
        label: "authName",
      },
      defKeys: [],
      roleId: "",
      roulesForm: {},
      editroulesForm: {},
      editRoulesVisiable: false,
      addRoulesVisiable: false,
      showSettingDialog: false,
      roulesRules: {
        roleName: [
          { required: true, message: "请输入角色名称", triggter: "bulr" },
          {
            min: 3,
            max: 10,
            message: "角色名称的长度在3~10之间",
            trigger: "blur",
          },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", triggter: "bulr" },
          {
            min: 3,
            max: 10,
            message: "角色描述的长度在2~10之间",
            trigger: "blur",
          },
        ],
      },
      editroulesRules: {
        roleName: [
          { required: true, message: "请输入角色名称", triggter: "bulr" },
          {
            min: 3,
            max: 10,
            message: "角色名称的长度在3~10之间",
            trigger: "blur",
          },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", triggter: "bulr" },
          {
            min: 3,
            max: 10,
            message: "角色描述的长度在2~10之间",
            trigger: "blur",
          },
        ],
      },
    };
  },
  methods: {
    async getRoleList() {
      const { data } = await this.$http.get("roles");
      if (data.meta.status !== 200) {
        return this.$message.error(data.meta.msg);
      }
      this.rolelist = data.data;
      console.log(data)
    },
    removeRightById(role, rightId) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          const { data } = await this.$http.delete(
            `roles/${role.id}/rights/${rightId}`
          );
          if (data.meta.status !== 200) {
            return this.$message.error(data.meta.msg);
          }
          this.$message({
            type: "success",
            message: "删除成功!",
          });
          role.children = data.data;
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    async showSetRightDialog(role) {
      this.roleId = role.id;
      const { data } = await this.$http.get("rights/tree");
      if (data.meta.status !== 200) {
        return this.$message.error(data.meta.msg);
      }
      this.rightslist = data.data;
      this.getLeafKeys(role, this.defKeys);
      this.setRightDialogVisible = true;
    },
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id);
      }
      node.children.forEach((item) => this.getLeafKeys(item, arr));
    },
    setRightDialogClosed() {
      this.defKeys = [];
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ];
      const idStr = keys.join(",");
      const { data } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idStr,
      });
      if (data.meta.status !== 200) {
        return this.$message.error(data.meta.msg);
      }
      this.$message.success("分配权限成功！");
      this.setRightDialogVisible = false;
      this.getRoleList();
    },
    editRoulesForm: async function(id) {
      console.log(id)
      this.editRoulesVisiable = true;
      const { data: result } = await this.$http.get("roles/" + id);
      console.log(result)
      this.editroulesForm = result.data;
    },
    DeleteRoules: async function(id) {
      const rerult = await this.$confirm(
        "此操作将永久删除用户,是否继续",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      //    如果用户确认删除则返回值是一个confirm字符串
      // 如果用户取消删除返回值是一个cancel字符串
      if (rerult !== "confirm") {
        return this.$message.info("已取消删除用户");
      }
      const { data: res } = await this.$http.delete("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除用户失败!");
      }
      this.$message.success("删除用户成功!");
      this.getRoleList();
    },
    editRoules: function() {
      // 表单再效验
      this.$refs.editRoulesRef.validate(async (val) => {
        if (!val) {
          return this.$message.error("输入信息有误!");
        }
        const { data: result } = await this.$http.put(
          "roles/" + this.editroulesForm.roleId,
          {
            roleName: this.editroulesForm.roleName,
            roleDesc: this.editroulesForm.roleDesc,
          }
        );
        if (result.meta.status !== 200) {
          return this.$message.error("修改角色失败!");
        }
        this.$message.success("修改用户成功!");
        this.editRoulesVisiable = false;
        this.getRoleList();
      });
    },
    addRoules: function() {
      this.$refs.addRoulesRef.validate(async (val) => {
        console.log(val);
        if (!val) {
          return this.$message.error("信息填写有误!");
        }

        const { data: result } = await this.$http.post(
          "roles",
          this.roulesForm
        );
        if (result.meta.status !== 201) {
          this.$message.error("添加用户失败");
        }
        this.$message.success("添加用户成功");
        this.addRoulesVisiable = false;
        // 重新刷新用户列表
        this.getRoleList();
      });
    },
    addrouleClosed: function() {
      this.$refs.addRoulesRef.resetFields();
    },
    editrouleClosed: function() {
      this.$refs.editRoulesRef.resetFields();
    },
  },
  created() {
    this.getRoleList();
  },
};
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bd-top {
  border-top: 1px solid #eee;
}

.bd-bottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
