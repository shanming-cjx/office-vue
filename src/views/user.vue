<template>
	<div v-if="isAuth(['ROOT', 'USER:SELECT'])">
    <!--  搜索条件栏  -->
		<el-form :inline="true" :model="dataForm" :rules="dataRule" ref="dataForm">
			<el-form-item prop="name"><!--  按姓名搜索+前端校验  -->
				<el-input
					v-model="dataForm.name"
					placeholder="姓名"
					size="medium"
					class="input"
					clearable="clearable"
				/>
			</el-form-item>
			<el-form-item><!--  性别  -->
				<el-select v-model="dataForm.sex" class="input" placeholder="性别" size="medium" clearable="clearable">
					<el-option label="男" value="男" />
					<el-option label="女" value="女" />
				</el-select>
			</el-form-item>
			<el-form-item><!--  权限角色：从数据库动态查询  -->
				<el-select v-model="dataForm.role" class="input" placeholder="角色" size="medium" clearable="clearable">
					<el-option v-for="one in roleList" :label="one.roleName" :value="one.roleName" />
				</el-select>
			</el-form-item>
			<el-form-item><!--  部门：从数据库动态查询  -->
				<el-select
					v-model="dataForm.deptId"
					class="input"
					placeholder="部门"
					size="medium"
					clearable="clearable"
				>
					<el-option v-for="one in deptList" :label="one.deptName" :value="one.id" />
				</el-select>
			</el-form-item>
			<el-form-item><!--  就职状态  -->
				<el-select
					v-model="dataForm.status"
					class="input"
					placeholder="状态"
					size="medium"
					clearable="clearable"
				>
					<el-option label="在职" value="1" />
					<el-option label="离职" value="2" />
				</el-select>
			</el-form-item>
			<el-form-item><!--  操作  -->
				<el-button size="medium" type="primary" @click="searchHandle()">查询</el-button>
				<el-button
					size="medium"
					type="primary"
					:disabled="!isAuth(['ROOT', 'USER:INSERT'])"
					@click="addHandle()"
				>
					新增
				</el-button>
				<el-button
					size="medium"
					type="danger"
					:disabled="!isAuth(['ROOT', 'USER:DELETE'])"
					@click="deleteHandle()"
				>
					批量删除
				</el-button>
			</el-form-item>
		</el-form>

    <!--  数据区  -->
    <!-- border 数据带边框；dataListLoading 加载动画；selectionChangeHandle 多选 -->
		<el-table
			:data="dataList"
			border
			v-loading="dataListLoading"
			@selection-change="selectionChangeHandle"
			cell-style="padding: 4px 0"
			style="width: 100%;"
			size="medium"
		>
			<el-table-column type="selection" header-align="center" align="center" width="50"/><!--  多选列  -->
			<el-table-column type="index" header-align="center" align="center" width="100" label="序号"><!--  序号列，查询结果的顺序非id主键  -->
				<template #default="scope">
					<span>{{ (pageIndex - 1) * pageSize + scope.$index + 1 }}</span>
				</template>
			</el-table-column>
			<el-table-column prop="name" header-align="center" align="center" min-width="100" label="姓名" />
			<el-table-column prop="sex" header-align="center" align="center" min-width="60" label="性别" />
			<el-table-column prop="tel" header-align="center" align="center" min-width="130" label="电话" />
			<el-table-column
				prop="email"
				header-align="center"
				align="center"
				min-width="240"
				label="邮箱"
				:show-overflow-tooltip="true"
			/><!--  show-overflow-tooltip：多余的内容悬停显示  -->
			<el-table-column prop="hiredate" header-align="center" align="center" min-width="130" label="入职日期" />
			<el-table-column
				prop="roles"
				header-align="center"
				align="center"
				min-width="150"
				label="角色"
				:show-overflow-tooltip="true"
			/>
			<el-table-column prop="dept" header-align="center" align="center" min-width="120" label="部门" />
			<el-table-column prop="status" header-align="center" align="center" min-width="100" label="状态" />
			<el-table-column header-align="center" align="center" width="150" label="操作"><!--  数据区操作  -->
				<template #default="scope">
					<el-button
						type="text"
						size="medium"
						v-if="isAuth(['ROOT', 'USER:UPDATE'])"
						@click="updateHandle(scope.row.id)"
					>
						修改
					</el-button>
					<el-button
						type="text"
						size="medium"
						v-if="isAuth(['ROOT', 'USER:UPDATE'])"
						:disabled="scope.row.status == '离职' || scope.row.root"
						@click="dimissHandle(scope.row.id)"
					>
						离职
					</el-button>
					<el-button
						type="text"
						size="medium"
						:disabled="scope.row.root"
						v-if="isAuth(['ROOT', 'USER:DELETE'])"
						@click="deleteHandle(scope.row.id)"
					>
						删除
					</el-button>
				</template>
			</el-table-column>
		</el-table>
    <!--  分页  -->
		<el-pagination
			@size-change="sizeChangeHandle"
			@current-change="currentChangeHandle"
			:current-page="pageIndex"
			:page-sizes="[10, 20, 50]"
			:page-size="pageSize"
			:total="totalCount"
			layout="total, sizes, prev, pager, next, jumper"
		></el-pagination>
		<add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="loadDataList"></add-or-update>
		<dimiss v-if="dimissVisible" ref="dimiss" @refreshDataList="loadDataList"></dimiss>
	</div>
</template>

<script>
import AddOrUpdate from './user-add-or-update.vue';
import Dimiss from './dimiss.vue';
export default {
	components: {
		AddOrUpdate,
		Dimiss
	},
	data() {
		return {
			dataForm: {//搜索栏数据
				name: '',
				sex: '',
				role: '',
				deptId: '',
				status: ''
			},
			dataList: [],//用户表格数据
			roleList: [],//搜索栏的下拉框角色数据
			deptList: [],//搜索栏的下拉框部门数据
			pageIndex: 1,//当前页码
			pageSize: 10,//每页条数默认10
			totalCount: 0,//总条数
			dataListLoading: false,//加载动画
			dataListSelections: [],//多选
			addOrUpdateVisible: false,//新增修改弹窗
			dimissVisible: false,//离职弹窗
			dataRule: {
				name: [{ required: false, pattern: '^[\u4e00-\u9fa5]{1,10}$', message: '姓名格式错误' }]
			}
		};
	},
	methods: {
    //加载数据区数据
    loadDataList() {
      this.dataListLoading = true;
      let data = {//封装查询条件数据
        page: this.pageIndex,
        length: this.pageSize,
        name: this.dataForm.name,
        sex: this.dataForm.sex,
        role: this.dataForm.role,
        deptId: this.dataForm.deptId,
        status: this.dataForm.status
      };
      this.$http('user/searchUserByPage', 'POST', data, true, resp => {
        let pageData = resp.pageData;//拿得到响应结果的数据pageData
        let list = pageData.list;
        for (let one of list) {//循环替换每个元素的状态
          if (one.status == 1) {
            one.status = '在职';
          } else if (one.status == 2) {
            one.status = '离职';
          }
        }
        this.dataList = list;//导入table数据源
        this.totalCount = pageData.totalCount;
        this.dataListLoading = false;
      });
    },
    //更改每页总条数
    sizeChangeHandle(val) {
      this.pageSize = val;
      //更改每页显示记录数量后，都从第一页开始查询
      this.pageIndex = 1;
      this.loadDataList();
    },
    //更改当前页码
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.loadDataList();
    },
    //搜索栏条件查询，搜索按钮
    searchHandle() {
      this.$refs['dataForm'].validate(valid => {
        if(valid){
          //清理页面上的表单验证结果
          this.$refs['dataForm'].clearValidate();
          //如果当前页面不是第一页，则跳转到第一页显示查询的结果
          if (this.pageIndex != 1) {
            this.pageIndex = 1;
          }
          this.loadDataList();
        }else {
          return false;
        }
      });
    },
    //查询下拉框角色列表
		loadRoleList() {
			let that = this;
			that.$http('role/searchAllRole', 'GET', null, true, resp => {
				that.roleList = resp.list;
			});
		},
    //查询下拉框部门列表
		loadDeptList() {
			let that = this;
			that.$http('dept/searchAllDept', 'GET', null, true, resp =>  {
				that.deptList = resp.list;
			});
		},
    //多选
		selectionChangeHandle(val) {
			this.dataListSelections = val;
		}
	},
	created() {
		this.loadRoleList();//加载搜索栏的下拉框角色数据
		this.loadDeptList();//加载搜索栏的下拉框部门数据
    this.loadDataList();//加载数据区数据
	}
};
</script>

<style lang="less" scoped="scoped"></style>
