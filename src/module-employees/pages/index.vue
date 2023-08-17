<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card>
        <span class="seleInfo">

        </span>
        <div>
          <el-form :model="requestParameters" label-width="30px">
            <el-input v-model="requestParameters.username" placeholder="姓名" clearable style="width: 200px"></el-input>
            <el-input v-model="requestParameters.mobile" placeholder="手机号" clearable style="width: 200px"></el-input>
            <el-input v-model="requestParameters.workNumber" placeholder="工号" clearable style="width: 200px"></el-input>
            <el-button type="primary" icon="el-icon-search" @click="doQuery">查询</el-button>
            <el-button type="primary" icon="el-icon-plus" @click="handlAdd">新增员工</el-button>
          </el-form>
        </div>
      </el-card>
      <el-card shadow="never" class="boxMar">
        <el-table :data="dataList" fit style="width: 100%;" border>
          <el-table-column type="index" :index="1" label="序号" width="150"></el-table-column>
          <el-table-column sortable prop="username" label="姓名" width="150"></el-table-column>
          <el-table-column sortable prop="mobile" label="手机号" width="150"></el-table-column>
          <el-table-column sortable prop="workNumber" label="工号" width="120"></el-table-column>
          <el-table-column sortable prop="departmentName" label="部门" width="200"></el-table-column>
          <el-table-column fixed="right" label="操作" align="center" width="220">
            <template slot-scope="scope">
              <router-link :to="{'path':'/employees/details/' + scope.row.id}"
                           class="el-button el-button--text el-button--small">
                查看
              </router-link>
              <el-button @click="handleRole(scope.row)" type="text" size="small">分配角色</el-button>
              <el-button v-if="show('point-user-delete')" @click="handleDelete(scope.row)" type="text" size="small">删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <!-- 分页 -->
        <div class="pagination">
          <PageTool :paginationPage="requestParameters.page" :paginationPagesize="requestParameters.size"
                    :total="counts" @pageChange="handleCurrentChange" @pageSizeChange="handleSizeChange">
          </PageTool>
        </div>
        <!-- end -->
        <!-- 新增员工弹层 -->
        <component v-bind:is="employeesAdd" ref="addUser" @doQuery="doQuery"></component>
        <!--分配角色组件 -->
        <component v-bind:is="addRole" ref="addRole"></component>
      </el-card>
    </div>
  </div>
</template>

<script>
  import constantApi from '@/api/constant/employees'
  import {list, remove} from "@/api/base/users"
  import PageTool from './../../components/page/page-tool'
  import employeesAdd from './../components/add'
  import addRole from './../components/addRole'
  import {hasPermissionPoint} from '@/utils/permission'

  export default {
    name: 'employeesList',
    components: {
      PageTool, employeesAdd, addRole
    },
    data() {
      return {
        employeesAdd: 'employeesAdd',
        addRole: 'addRole',
        baseData: constantApi,
        dataList: [],
        counts: '',
        requestParameters: {
          page: 1,
          size: 10,
          mobile: '',
          username: '',
          workNumber: ''
        }
      }
    },
    methods: {
      show(name) {
        return hasPermissionPoint(name)
      },
      // 业务方法
      doQuery(params) {
        list(this.requestParameters)
          .then(res => {
            this.dataList = res.data.data.rows
            this.counts = res.data.data.total
          })
      },
      // 每页显示信息条数
      handleSizeChange(size) {
        this.requestParameters.size = size
        if (this.requestParameters.page === 1) {
          this.doQuery(this.requestParameters)
        }
      },
      // 进入某一页
      handleCurrentChange(val) {
        this.requestParameters.page = val
        this.doQuery()
      },
      // 添加新员工
      handlAdd() {
        this.$refs.addUser.dialogFormVisible = true
      },
      // 删除
      handleDelete(item) {
        this.$confirm(
          `本次操作将删除${item.username}删除后账号将不可恢复，您确认删除吗？`, {
            type: 'warning'
          }
        ).then(() => {
          remove({id: item.id})
            .then(res => {
              this.$message({message: res.data.message, type: res.data.success ? 'success' : 'error'})
              if (res.data.success) {
                this.doQuery();
              }
            })
        })
      },
      handleRole(item) {
        this.$refs.addRole.toAssignPrem(item.id)
      },
    },
    // 创建完毕状态
    created: function () {
      this.doQuery()
    },
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .iconInfo {
    .fa {
      color: #999;
      font-size: 14px;
      cursor: pointer;
    }

    a {
      padding: 0 5px;
    }
  }

  .serachInput {
    .el-input--medium {
      width: 150px;

      .el-input__inner {
      }
    }
  }

  .serachInput .el-input--medium .el-input__inner {
    height: 26px;
    line-height: 26px;
  }
</style>
