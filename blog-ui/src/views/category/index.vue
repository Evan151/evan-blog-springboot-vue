<template>
    <div class="app-container">
        <el-form :inline="true" :model="query" size="mini" class="demo-form-inline">
            <el-form-item label="分类名称">
                <el-input v-model.trim="query.name"></el-input>
            </el-form-item>
            <el-form-item label="状态">
                <el-select clearable filterable v-model="query.status">
                    <el-option v-for="item in statusOptions" :key="item.code" :label="item.status" :value="item.code"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button icon="el-icon-search" type="primary" @click="queryData">查询</el-button>
            </el-form-item>
            <el-form-item>
                <el-button icon="el-icon-refresh" @click="reload">重置</el-button>
            </el-form-item>
            <el-form-item>
                <el-button icon="el-icon-circle-plus-outline" type="primary" @click="openAdd">新增</el-button>
            </el-form-item>
        </el-form>
        <el-table
                :data="list"
                stripe
                border
                highlight-current-row
                style="width: 100%">
            <el-table-column align="center" type="index" label="序号" width="60"></el-table-column>
            <el-table-column align="center" prop="name" label="分类名称"></el-table-column>
            <el-table-column align="center" prop="sort" label="排序"></el-table-column>
            <el-table-column align="center" prop="remark" label="备注"></el-table-column>
            <el-table-column align="center" prop="status" label="状态">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.status | statusFilter">
                        {{ scope.row.status ? '正常':'禁用'}}
                    </el-tag>

                </template>

            </el-table-column>
            <el-table-column align="center" label="操作">
                <template slot-scope="scope">
                    <el-button size="mini" type="success"
                               @click="handleEdit(scope.row.id)">编辑
                    </el-button>
                    <el-button
                            size="mini"
                            type="danger"
                            @click="handleDelete(scope.row.id)">删除
                    </el-button>
                </template>
            </el-table-column>


        </el-table>
        <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="page.current"
                :page-sizes="[10, 20, 30, 40]"
                :page-size="page.size"
                layout="total, sizes, prev, pager, next, jumper"
                :total="page.total">
        </el-pagination>
        <edit :formData="edit.formData" :title="edit.title" :visible="edit.visible" :remote-close="remoteClose"></edit>
    </div>

</template>


<script>

import categoryApi from "@/api/category";
import edit from "@/views/category/edit";


const statusOptions = [

    {code: '1', status: '正常'},
    {code: '0', status: '禁用'},
]

export default {
    components: {
        edit
    },
    data() {
        return {
            query: {},
            page: {
                current: 1,
                size: 20,
                total: 0
            },
            list: [],
            statusOptions,
            edit: {
                title: '',
                visible: false,
                formData: {}
            }
        }

    },
    filters: {
        statusFilter(status) {
            const statusMap = {0: 'danger', 1: 'info'}
            return statusMap[status]
        }

    },

    created() {
        this.fetchData()
    },

    methods: {
        fetchData() {
            categoryApi.getList(this.query, this.page.current, this.page.size).then(res => {
                console.log(res.data)
                this.page.total = res.data.total
                this.list = res.data.records;

            })

        },
        handleEdit(id) {
            categoryApi.getById(id).then(response => {
                if (response.code === 20000) {
                    this.edit.formData = response.data
                    this.edit.title = '编辑';
                    this.edit.visible = true;
                }
            })
        },
        handleDelete(id) {
            this.$confirm('此操作将永久删除该条记录, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                categoryApi.deleteById(id).then(response => {
                    this.$message({
                        type: response.code === 20000 ? 'success' : 'error',
                        message: response.message
                    })
                    this.fetchData();
                })
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });
            });
        },

        handleSizeChange(val) {
            this.page.size = val
            this.fetchData()
        },
        handleCurrentChange(val) {
            this.page.current = val
            this.fetchData()
        },

        queryData() {
            this.page.current = 1
            this.fetchData()
        },
        reload() {
            this.query = {}
            this.fetchData()

        },
        remoteClose() {
            this.edit.formData = {}
            this.edit.visible = false
            this.fetchData()
        },
        openAdd() {
            this.edit.visible = true;
            this.edit.title = "新增"
        }

    },

}

</script>

<style scoped>

</style>
