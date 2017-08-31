
<template>
    <div class="clear">
        <el-input placeholder="请选择关键字" icon="search" class="searchbox" v-model="seachbyId">
        </el-input>
        <el-button type="primary" class="addbtn" @click="addOrder">新 增 订 单</el-button>
        <el-table :data="filtermatchdata" border style="width: 100%;" class="ordertable">
            <el-table-column type="expand">
                <template scope="scope">
                    <el-form label-position="left" inline class="demo-table-expand">
                        <el-form-item label="工程名称">
                            <span>{{ scope.row.workname}}</span>
                        </el-form-item>
                        <el-form-item label="材料名称">
                            <span>{{ scope.row.workname}}</span>
                        </el-form-item>
                        <el-form-item label="材料直径">
                            <span>{{ scope.row.workname}}</span>
                        </el-form-item>
                        <el-form-item label="材料长度">
                            <span>{{ scope.row.workname}}</span>
                        </el-form-item>
                        <el-form-item label="材料重量">
                            <span>{{ scope.row.workname}}</span>
                        </el-form-item>
                        <el-form-item label="总价">
                            <!-- <span>{{ scope.row.totalprice}}</span> -->
                            <span>{{ scope.row.amount*scope.row.price}}</span>
                        </el-form-item>

                        <el-form-item label="数量">
                            <span>{{ scope.row.amount}}</span>

                        </el-form-item>
                        <el-form-item label="单价">

                            <span>{{ scope.row.price}}</span>
                        </el-form-item>
                        <el-form-item label="简图">

                            <span class="imgstyle">
                                <lazy-component @show="handler">
                                    <img v-bind:src="scope.row.pic" alt="" srcset="">
                                </lazy-component>
                            </span>

                        </el-form-item>
                    </el-form>
                </template>
            </el-table-column>
            <el-table-column label="订单编号">
                <template scope="scope">
                    <span style="margin-left: 10px;">{{ scope.row.id }}</span>
                </template>
            </el-table-column>
            <el-table-column label="公司名称">
                <template scope="scope">
                    <span> {{ scope.row.companyname}}</span>
                </template>
            </el-table-column>

            <el-table-column label="到期日">
                <template scope="scope">
                    <span>{{ scope.row.targettime}}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template scope="scope">
                    <el-button @click="updatestatus(scope.$index, scope.row)" type="text" v-if="scope.row.status!='订单出库'">更新进度</el-button>
                    <span v-if="scope.row.status=='订单出库'">订单已出库</span>
                    <el-button @click="printEdit(scope.$index, scope.row)" type="text">订单打印</el-button>

                </template>
            </el-table-column>
        </el-table>
        <el-pagination v-bind:current-Page="pageIndex" v-bind:page-size="pageSize" :total="total" layout="total,sizes,prev,pager,next,jumper" v-bind:page-sizes="pageSizes" v-on:size-change="sizeChange" v-on:current-change="pageIndexChange" class="pagination">

        </el-pagination>

        <!--订单进度-->
        <el-dialog title="订单进度" :visible.sync="dialogFormVisible">
            <el-steps :space="100" :active="active" finish-status="success">
                <el-step title="订单下达"></el-step>
                <el-step title="生产完成"></el-step>
                <el-step title="订单出库"></el-step>
            </el-steps>
            <!-- <el-form :model="form">
                            <el-form-item label="活动名称" :label-width="formLabelWidth">
                                <el-input v-model="form.name" auto-complete="off"></el-input>
                            </el-form-item>
                            <el-form-item label="活动区域" :label-width="formLabelWidth">
                                <el-select v-model="form.region" placeholder="请选择活动区域">
                                    <el-option label="区域一" value="shanghai"></el-option>
                                    <el-option label="区域二" value="beijing"></el-option>
                                </el-select>
                            </el-form-item>
                        </el-form> -->
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="update">下一步</el-button>
            </div>
        </el-dialog>

    </div>
</template>

<script>
// import this.servicerurl from '../js/host.js'
var STORAGE_KEY = 'orderList';
var orderStorage = {
    save: function(data) {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(data))
    },
    fetch: function() {
        return JSON.parse(localStorage.getItem(STORAGE_KEY) || []);
    }
};


export default {
    data() {
        return {
            dialogFormVisible: false,
            selectedTable: [],
            pageIndex: 0,
            pageSize: 10,
            total: 0,
            pageSizes: [10, 20, 50, 100],
            totaldata: [],
            tableData: [],
            seachbyId: '',
            matchdata: [],
            id: '',
            active: 1

        }
    },
    mounted: function() {
        this.$http.get(this.servicerurl + '/order', {}, {
            headers: {},
            emulateJSON: true
        }).then(function(response) {

            orderStorage.save(response.data);
            this.tableData = orderStorage.fetch();
            console.log(response.data)
        }, function(response) {
            console.log(response)
        })

    },
    watch: {
        tableData: {
            handler() {
                orderStorage.save(this.tableData);
                this.total = this.filtermatchdata.length;
            },
            deep: true
        }
    },
    computed: {
        filtermatchdata: function() {
            let matchdata = this.searchFilter('id', this.seachbyId, this.tableData);
            return matchdata;
        },
        totalPrice: function() {
            console.log(this.orderdetail1);
            row.totalprice = Number(row.amount1) * Number(row.price1);
            return this.orderdetail1.totalprice.toString();
        }
    },
    methods: {
        updatestatus: function($index, row) {
            if (row.status == "生产完成") { this.active = 2; }
            else if (row.status == "订单出库") { this.active = 3; }
            else this.active = 1;
            this.selectedTable = row;
            this.dialogFormVisible = true;
            console.log(this.selectedTable)
        },
        update: function() {
            //更改订单状态

            if (this.selectedTable.status == "生产完成") {
                this.selectedTable.status ="订单出库"
            }
            else { this.selectedTable.status = "生产完成" }
            //    console.log(this.selectedTable)
            this.$http.put(this.servicerurl + '/order' + '/' + this.selectedTable.id, this.selectedTable, {
                headers: {},
                emulateJSON: true
            }).then(function(response) {
            this.dialogFormVisible=false;
                this.$http.get(this.servicerurl + '/order', {
                    pageIndex: this.pageIndex,
                    pageSize: this.pageSize
                }, {
                        headers: {},
                        emulateJSON: true
                    }).then(function(response) {
                        console.log(response.data);


                        this.$message({
                            showClose: true,
                            message: '更新成功',
                            type: 'success'
                        })
                    })
                console.log(response.data);

            }, function(response) {
                console.log(response);
            })

        },
        handler: function() {
            console.log('this component is showing')
        },

        sizeChange: function(pageSize) {
            this.pageSize = pageSize;
            this.fetchData();
        },
        pageIndexChange: function(pageIndex) {
            this.pageIndex = pageIndex;
            this.fetchData();
        },
        fetchData: function() {
            this.loading = true;
            this.$http.get(this.servicerurl + '/order', {
                pageIndex: this.pageIndex,
                pageSize: this.pageSize
            }, {
                    headers: {},
                    emulateJSON: true
                }).then(function(response) {
                    orderStorage.save(response.data);
                    this.tableData = orderStorage.fetch();
                    this.loading = false;
                    console.log(response.data)
                }, function(response) {
                    console.log(response)
                })
        },
        handleDelete: function($index, row) {
            var id1 = row.id;
            //resource
            var resource = this.$resource(this.servicerurl + '/order/{id}');
            resource.delete({ id: id1 }).then(response => {
                // success callback
                this.$http.get(this.servicerurl + '/order', {}, {
                    headers: {},
                    emulateJSON: true
                }).then(function(response) {
                    //get again
                    orderStorage.save(response.data);
                    this.tableData = response.data;
                    console.log(response.data);
                }, function(response) {
                    console.log(response)
                })
                console.log(id1);
                console.log(response.data);
            }, response => {
                // error callback
                console.log(id1);
                console.log(response);
            })


        },
        searchFilter: function(prop, key, arr) {
            if (!key) {
                return arr;
            }
            arr = arr.filter((row) => {
                if (row[prop].toString().indexOf(key) !== -1) {
                    return true
                }
            })
            if (key !== this[prop]) {
                this.currentPage = 0
                this[prop] = key
            }
            return arr;
        },
        addOrder: function() {
            this.$router.push({ path: '/order/add' })
        },
        printEdit: function($index, row) {
            // var datatemp =
            //     {
            //         id: row.id,
            //         companyname: row.companyname,
            //         workname: row.workname,
            //         totalprice: row.totalprice,
            //         amount: row.amount,
            //         pic: row.pic,
            //         price: row.price,
            //         targettime: row.targettime,
            //     };
            // console.log(datatemp);
            // localStorage.setItem("orderTemp", JSON.stringify(datatemp));
            // this.$router.push({ path: '/order/modify' })
            // console.log("print")
            window.print();
        }

    }
}
</script>
<style>
.demo-table-expand {
    font-size: 0;
}

.demo-table-expand label {
    width: 90px;
    color: #99a9bf;
}

.demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
}

.ordertable {

    top: 20px !important;
    font-size: 12px;
    margin-left: 20px;
}

.searchbox {

    width: 30%;
    margin-left: 20px !important;
}

.addbtn {
    background-color: #ff9800 !important;
    float: right;

    margin-right: 20px;
}

.clear {
    clear: both
}

.dangeraction {
    color: red !important;
}

.pagination {
    margin-top: 30px;
    margin-left: 40%;
}

.imgstyle img {
    width: 95px;
}
</style>
