<template>
  <!-- 行数据id存在则dataForm.id>0;行数据id不存在则dataForm.id=0;0为false，不存在则新增，反之为修改 -->
	<el-dialog
		:title="!dataForm.id ? '新增' : '修改'"
		v-if="isAuth(['ROOT', 'USER:INSERT', 'USER:UPDATE'])"
		:close-on-click-modal="false"
		v-model="visible"
		width="450px"
	>
		<el-form :model="dataForm" ref="dataForm" :rules="dataRule" label-width="80px">
			<el-form-item label="用户名" prop="username">
				<el-input v-model="dataForm.username" size="medium" clearable />
			</el-form-item>
			<el-form-item label="密码" prop="password">
				<el-input type="password" v-model="dataForm.password" size="medium" clearable />
			</el-form-item>
			<el-form-item label="姓名" prop="name">
				<el-input v-model="dataForm.name" size="medium" clearable />
			</el-form-item>
			<el-form-item label="性别" prop="sex">
				<el-select v-model="dataForm.sex" size="medium" style="width: 100%;" clearable>
					<el-option label="男" value="男"></el-option>
					<el-option label="女" value="女"></el-option>
				</el-select>
			</el-form-item>
			<el-form-item label="电话" prop="tel">
				<el-input v-model="dataForm.tel" size="medium" clearable />
			</el-form-item>
			<el-form-item label="邮箱" prop="email">
				<el-input v-model="dataForm.email" size="medium" clearable />
			</el-form-item>
			<el-form-item label="入职日期" prop="hiredate">
				<el-date-picker
					v-model="dataForm.hiredate"
					type="date"
					placeholder="选择日期"
					size="medium"
					:editable="false"
					format="YYYY-MM-DD"
					value-format="YYYY-MM-DD"
					style="width: 100%;"
				/>
			</el-form-item>
			<el-form-item label="角色" prop="role">
				<el-select
					v-model="dataForm.role"
					size="medium"
					placeholder="选择角色"
					style="width: 100%;"
					multiple
					clearable
				>
					<el-option
						v-for="one in roleList"
						:key="one.id"
						:label="one.roleName"
						:value="one.id"
						:disabled="one.roleName == '超级管理员'"
					></el-option>
				</el-select>
			</el-form-item>
			<el-form-item label="部门" prop="deptId">
				<el-select
					v-model="dataForm.deptId"
					size="medium"
					placeholder="选择部门"
					style="width: 100%;"
					clearable
				>
					<el-option v-for="one in deptList" :key="one.id" :label="one.deptName" :value="one.id" />
				</el-select>
			</el-form-item>
		</el-form>
		<template #footer>
			<span class="dialog-footer">
				<el-button size="medium" @click="visible = false">取消</el-button>
				<el-button type="primary" size="medium" @click="dataFormSubmit">确定</el-button>
			</span>
		</template>
	</el-dialog>
</template>

<script>
import dayjs from 'dayjs';
export default {
	data() {
		return {
			visible: false,
			dataForm: {
				id: null,
				username: null,
				password: null,
				name: null,
				sex: null,
				tel: null,
				email: null,
				hiredate: new Date(),
				role: null,
				deptId: null,
				status: 1
			},
			roleList: [],
			deptList: [],
			dataRule: {
				username: [{ required: true, pattern: '^[a-zA-Z0-9]{5,20}$', message: '用户名格式错误' }],
				password: [{ required: true, pattern: '^[a-zA-Z0-9]{6,20}$', message: '密码格式错误' }],
				name: [{ required: true, pattern: '^[\u4e00-\u9fa5]{2,10}$', message: '姓名格式错误' }],
				sex: [{ required: true, message: '性别不能为空' }],
				tel: [{ required: true, pattern: '^1\\d{10}$', message: '电话格式错误' }],
				email: [
					{
						required: true,
						pattern: '^([a-zA-Z]|[0-9])(\\w|\\-)+@[a-zA-Z0-9]+\\.([a-zA-Z]{2,4})$',
						message: '邮箱格式错误'
					}
				],
				hiredate: [{ required: true, trigger: 'blur', message: '入职日期不能为空' }],
				role: [{ required: true, message: '角色不能为空' }],
				deptId: [{ required: true, message: '部门不能为空' }],
				status: [{ required: true, message: '状态不能为空' }]
			}
		};
	},
	methods: {
		init(id) {//获取行数据的id
			this.dataForm.id = id || 0;//若行数据的id存在，赋值给表单的id；否则赋0
			this.visible = true;
			this.$nextTick(() => {
				this.$refs['dataForm'].resetFields();
				this.$http('role/searchAllRole', 'GET', null, true, resp => {
					this.roleList = resp.list;
				});
				this.$http('dept/searchAllDept', 'GET', null, true, resp => {
					this.deptList = resp.list;
				});
				if (this.dataForm.id) {
					this.$http('user/searchById', 'POST', { userId: id }, true, resp => {
						this.dataForm.username = resp.username;
						this.dataForm.password = resp.password;
						this.dataForm.name = resp.name;
						this.dataForm.sex = resp.sex;
						this.dataForm.tel = resp.tel;
						this.dataForm.email = resp.email;
						this.dataForm.hiredate = resp.hiredate;
						this.dataForm.role = JSON.parse(resp.role);
						this.dataForm.deptId = resp.deptId;
						this.dataForm.status = resp.status;
					});
				}
			});
		},
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if(valid){
          let data = {
            userId: this.dataForm.id,
            username: this.dataForm.username,
            password: this.dataForm.password,
            name: this.dataForm.name,
            sex: this.dataForm.sex,
            tel: this.dataForm.tel,
            email: this.dataForm.email,
            hiredate: dayjs(this.dataForm.hiredate).format('YYYY-MM-DD'),//转选择器对选为字符串
            role: this.dataForm.role,
            deptId: this.dataForm.deptId,
            status: this.dataForm.status
          };
          //模板字符串用``不是'', 变量使用${}。
          this.$http(`user/${!this.dataForm.id?'insert':'update'}`,"POST",data,true,resp => {
            if(resp.rows==1){
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1200
              });
              this.visible=false
              this.$emit('refreshDataList');//调用其他页面函数
            }
            else{
              this.$message({
                message: '操作失败',
                type: 'error',
                duration: 1200
              });
            }
          });
        }
      })
    },
	}
};
</script>

<style lang="less" scoped="scoped">
</style>
