<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.search"
        placeholder="请输入内容"
        clearable
        prefix-icon="el-icon-search"
        style="width: 200px;"
        class="filter-item"
        @keyup.enter.native="handleFilter"
        @clear="handleFilter"
      />
      <el-button-group>
        <el-button
          class="filter-item"
          type="primary"
          icon="el-icon-search"
          @click="handleFilter"
        >{{ "搜索" }}</el-button>
        <el-button
          v-if="permissionList.add"
          class="filter-item"
          type="success"
          icon="el-icon-edit"
          @click="handleCreate"
        >{{ "添加" }}</el-button>
      </el-button-group>
    </div>

    <el-table
      :data="list"
      v-loading="listLoading"
      border
      style="width: 100%"
      highlight-current-row
      @sort-change="handleSortChange"
    >
      <el-table-column label="名称" prop="name"></el-table-column>
      <el-table-column label="任务" prop="task"></el-table-column>
      <el-table-column label="参数" prop="args"></el-table-column>
      <el-table-column label="状态" prop="enabled">
        <template slot-scope="{ row }">
          <el-tag v-if="row.enabled" type="success">启用</el-tag>
          <el-tag v-else type="danger">禁用</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="开始时间" prop="start_time"></el-table-column>
      <el-table-column label="失效时间" prop="expires"></el-table-column>
      <el-table-column label="操作" align="center" width="260" class-name="small-padding fixed-width">
        <template slot-scope="{ row }">
          <el-button-group>
            <el-button
              v-if="permissionList.update"
              size="small"
              type="primary"
              @click="handleUpdate(row)"
            >{{ "编辑" }}</el-button>
            <el-button
              v-if="permissionList.del"
              size="small"
              type="danger"
              @click="handleDelete(row)"
            >{{ "删除" }}</el-button>
          </el-button-group>
        </template>
      </el-table-column>
    </el-table>
    <div class="table-pagination">
      <pagination
        v-show="total > 0"
        :total="total"
        :page.sync="listQuery.offset"
        :limit.sync="listQuery.limit"
        @pagination="getList"
      />
    </div>
    <el-dialog
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
      :close-on-click-modal="false"
    >
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="80px"
        style="width: 400px; margin-left:50px;"
      >
        <el-form-item label="名称" prop="name">
          <el-input v-model="temp.name" />
        </el-form-item>
        <el-form-item label="cron配置" prop="crontab">
          <el-select v-model="temp.crontab" clearable placeholder="请选择">
            <el-option
              v-for="item in crontab_list"
              :key="item.id"
              :label="item.name"
              :value="item.cron.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="任务" prop="task">
          <el-select v-model="temp.task" clearable placeholder="请选择">
            <el-option
              v-for="item in taskscript_list"
              :key="item.id"
              :label="item.name"
              :value="item.code"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="参数" prop="args">
          <el-input v-model="temp.args" />
        </el-form-item>
        <el-form-item label="状态" prop="enabled">
          <el-switch v-model="temp.enabled" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
        </el-form-item>
        <el-form-item label="开始时间" prop="start_time">
          <el-date-picker
            v-model="temp.start_time"
            type="datetime"
            placeholder="选择日期时间"
            align="right"
          ></el-date-picker>
        </el-form-item>
        <el-form-item label="过期时间" prop="expires">
          <el-date-picker v-model="temp.expires" type="datetime" placeholder="选择日期时间" align="right"></el-date-picker>
        </el-form-item>
        <el-form-item label="备注" prop="memo">
          <el-input v-model="temp.memo" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">{{ "取消" }}</el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
        >{{ "确定" }}</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { task, crontab, taskscript, auth } from "@/api/all";
import Pagination from "@/components/Pagination";
import {
  checkAuthAdd,
  checkAuthDel,
  checkAuthView,
  checkAuthUpdate
} from "@/utils/permission";

export default {
  name: "task",

  components: { Pagination },
  data() {
    return {
      operationList: [],
      permissionList: {
        add: false,
        del: false,
        view: false,
        update: false
      },
      list: [],
      total: 0,
      listLoading: true,
      loading: true,
      listQuery: {
        offset: 1,
        limit: 20,
        search: undefined,
        ordering: undefined
      },
      temp: {},
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "编辑",
        create: "添加"
      },
      rules: {
        name: [{ required: true, message: "请输入名称", trigger: "blur" }]
      },
      crontab_list: [],
      taskscript_list: []
    };
  },
  computed: {},
  created() {
    this.getMenuButton();
    this.getList();
    this.getCrontab();
    this.getScript();
  },
  methods: {
    checkPermission() {
      this.permissionList.add = checkAuthAdd(this.operationList);
      this.permissionList.del = checkAuthDel(this.operationList);
      this.permissionList.view = checkAuthView(this.operationList);
      this.permissionList.update = checkAuthUpdate(this.operationList);
    },
    getMenuButton() {
      auth
        .requestMenuButton("task")
        .then(response => {
          this.operationList = response.results;
        })
        .then(() => {
          this.checkPermission();
        });
    },
    getList() {
      this.listLoading = true;
      task.requestGet(this.listQuery).then(response => {
        this.list = response.results;
        this.total = response.count;
        this.listLoading = false;
      });
    },
    getCrontab() {
      crontab.requestGet().then(response => {
        this.crontab_list = response.results;
      });
    },
    getScript() {
      taskscript.requestGet().then(response => {
        this.taskscript_list = response.results;
      });
    },
    handleFilter() {
      this.getList();
    },
    handleSortChange(val) {
      if (val.order === "ascending") {
        this.listQuery.ordering = val.prop;
      } else if (val.order === "descending") {
        this.listQuery.ordering = "-" + val.prop;
      } else {
        this.listQuery.ordering = "";
      }
      this.getList();
    },
    resetTemp() {
      this.temp = {
        name: "",
        task: "",
        args: "[]",
        enabled: true,
        crontab: "",
        one_off: false,
        start_time: "",
        expires: "",
        memo: ""
      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.loading = false;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    createData() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.loading = true;
          task
            .requestPost(this.temp)
            .then(response => {
              this.dialogFormVisible = false;
              this.$notify({
                title: "成功",
                message: "创建成功",
                type: "success",
                duration: 2000
              });
              this.getList();
            })
            .catch(() => {
              this.loading = false;
            });
        }
      });
    },
    handleUpdate(row) {
      this.temp = row;
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.loading = true;
          task
            .requestPut(this.temp.id, this.temp)
            .then(() => {
              this.dialogFormVisible = false;
              this.$notify({
                title: "成功",
                message: "更新成功",
                type: "success",
                duration: 2000
              });
            })
            .catch(() => {
              this.loading = false;
            });
        }
      });
    },
    handleDelete(row) {
      this.$confirm("是否确定删除?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          task.requestDelete(row.id).then(() => {
            this.$message({
              message: "删除成功",
              type: "success"
            });
            this.getList();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>
