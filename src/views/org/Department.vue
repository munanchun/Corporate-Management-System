<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.name" placeholder="部门名称"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="search">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="departments" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="部门名称" width="120" sortable>
			</el-table-column>
      <el-table-column prop="sn" label="部门编号" width="120" sortable>
      </el-table-column>
      <el-table-column prop="state" label="部门状态" width="120" sortable>
        <template slot-scope="scope">
          <span style="color: #13ce66" v-if="scope.row.state==0">正常</span>
          <span style="color: #e64242" v-else>禁用</span>
        </template>
      </el-table-column>
      <el-table-column prop="manager.username" label="管理员" width="120" sortable>
      </el-table-column>
      <el-table-column prop="parent.name" label="上级部门" width="120" sortable>
      </el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange"
                     :current-page="currentPage"
                     :page-size="pageSize"
                     :total="totals"
                     style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog :title="title" :visible.sync="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="部门名称" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
        <el-form-item label="部门编号" prop="sn">
          <el-input v-model="editForm.sn" auto-complete="off"></el-input>
        </el-form-item>
				<el-form-item label="部门状态">
					<el-radio-group v-model="editForm.state">
						<el-radio class="radio" :label="1">禁用</el-radio>
						<el-radio class="radio" :label="0">正常</el-radio>
					</el-radio-group>
				</el-form-item>
        <el-form-item label="管理员" prop="manager">
          <el-input v-model="editForm.manager" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="上级部门" prop="parent">
          <el-input v-model="editForm.parent" auto-complete="off"></el-input>
        </el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<!--<el-dialog title="新增" visible.sync="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="姓名" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="性别">
					<el-radio-group v-model="addForm.sex">
						<el-radio class="radio" :label="1">男</el-radio>
						<el-radio class="radio" :label="0">女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="年龄">
					<el-input-number v-model="addForm.age" :min="0" :max="200"></el-input-number>
				</el-form-item>
				<el-form-item label="生日">
					<el-date-picker type="date" placeholder="选择日期" v-model="addForm.birth"></el-date-picker>
				</el-form-item>
				<el-form-item label="地址">
					<el-input type="textarea" v-model="addForm.addr"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>-->
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';
  import Axios from "axios";
  import {id} from "html-webpack-plugin/lib/chunksorter";

	export default {
		data() {
			return {
				filters: {
					name: ''
				},
        title:'',
				departments: [],
				totals: 0,
				currentPage:1,
        pageSize:5,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					id: 0,
          name: '',
          sn:'',
          dirPath:'',
          state:0,
          manager:null,
          parent:null
				},

				/*addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					sex: -1,
					age: 0,
					birth: '',
					addr: ''
				}*/

			}
		},
		methods: {
      search(){
        this.currentPage=1;
        this.getDepartments();
      },
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.currentPage = val;
				this.getDepartments();
			},
			//获取部门 列表
			getDepartments() {
				let para = {
					currentPage: this.currentPage,
          pageSize:this.pageSize,
					keyword: this.filters.name
				};
				this.listLoading = true;
				//NProgress.start();
        Axios.post("/dept",para).then(res=>{
          console.log(res);
          this.listLoading=false
          //给列表页数据赋值
          this.departments=res.data.rows;
          //给总条数赋值
          this.totals=res.data.totals;
        }) //res对象用来接收后端返回来的数据
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { id: row.id };
          Axios.delete("/dept/"+row.id).then(res=>{
            console.log(res)
            this.listLoading = false;
            this.$message({
              message: '删除成功',
              type: 'success'
            });
            this.getDepartments();
          })
				}).catch(() => {
          this.$message({
            message: '删除失败',
            type: 'error'
          });

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
        this.title="编辑"
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
			},
			//显示新增界面
			handleAdd: function () {

        this.title="新增"
				this.editFormVisible = true;
				this.editForm = {
					id:null,
          name: '',
          sn:'',
          dirPath:'',
          state:0,
					manager:null,
          parent:null,
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							Axios.put("/dept",para).then(res=>{
                console.log(res)
                this.editLoading=false;
                this.editFormVisible=false;
                if(res.data.success){
                  this.$message({
                    message:'操作成功！',
                    type:'success'
                  });
                  this.search()
                }else {
                  this.$message({
                    message:'操作失败！',
                    type:'error'
                  });
                }
              })
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							addUser(para).then((res) => {
								this.addLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['addForm'].resetFields();
								this.addFormVisible = false;
								this.getDepartments();
							});
						});
					}
				});
			},
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id)
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
          Axios.patch("/dept",ids).then(res=>{
            if (res.data.success){
              this.$message({
                message: '删除成功',
                type: 'success'
              });
            }
            this.listLoading = false;
            this.getDepartments()
          })
				}).catch(() => {
          this.$message({
            message: '删除失败',
            type: 'error'
          });
				});
			}
		},
		mounted() {
			this.getDepartments();
		}
	}

</script>

<style scoped>

</style>