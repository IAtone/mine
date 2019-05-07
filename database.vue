<template>
<!-- 资料库 -->
    <div class="database" @click="hideUl">
        <div class="file-msg">
            <div class="file">
                <div class="title">
                    <img src="../../static/img/file.png" alt="">
                    常用文件
                </div>
                <div class="file_content">
                    <ul>
                        <li v-for="(item, index) in aFile" :key="index">
                            <a href="">
                                <img src="../../static/img/word.png" alt="">
                                {{ item.name }}
                            </a>
                        </li>
                    </ul>
                </div>
            </div>
            <div class="msg">
                <div class="title">
                    <img src="../../static/img/msg.png" alt="" style="width: 20px;height: 20px;">
                    消息
                </div>
                <div class="msg_content">
                    <ul>
                        <li v-for="(item, index) in aMsg" :key="index">
                            <span>{{ item.time }}</span>
                            <p>{{ item.message }}</p>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="files">
            <el-tree :data="files" :props="defaultProps" @node-click="handleNodeClick"
            node-key="id" :default-expanded-keys="defaultArr" :default-checked-keys="defaultArr" highlight-current ref="tree" current-node-key>
                <span class="custom-tree-node" slot-scope="{ node, data }">
                    <span>
                        <img src="../../static/img/files.png" alt="" v-if="data.isfolder" class="folder">
                        <!-- <img src="../../static/img/word.png" alt="" v-else-if="data.name.endsWith('.docx')">
                        <img src="../../static/img/ppt.png" alt="" v-else-if="data.name.endsWith('.ppt')">
                        <img src="../../static/img/pdf.png" alt="" v-else-if="data.name.endsWith('.pdf')">
                        <img src="../../static/img/xlsx.png" alt="" v-else-if="data.name.endsWith('.xlsx')"> -->
                        {{ node.label }}
                    </span>
                </span>
            </el-tree>
        </div>
        <div class="wrapper">
            <div class="fileshow">
                <div class="top">
                    <div class="upload" @change="uploadFile($event)">
                        <input type="file" id="file">
                        <span>上传文件</span>
                    </div>
                    <span class="newfile" @click="showNewFile">新建文件夹</span>
                    <el-button type="danger" icon="el-icon-delete" size="mini" v-show="totalItem > 0" @click="deleteAll(idarr)">批量删除</el-button>
                    <div class="search">
                        <input type="text" placeholder="搜索您的文件" v-model="keyword" @keyup.enter="search">
                        <i class="el-icon-search" @click="search"></i>
                    </div>
                </div>
                <div class="content">
                    <div class="title">
                        <div class="breadcrumb">
                            <el-breadcrumb separator-class="el-icon-arrow-right">
                                <el-breadcrumb-item class="is-link" :to="{ path: '/' }">
                                    <span class="return" @click="returnTop" v-if="newid">返回上一级</span>全部文件
                                </el-breadcrumb-item>
                                <el-breadcrumb-item v-for="(item, index) in crumbArr" :key="index">{{ item }}</el-breadcrumb-item>
                                <el-breadcrumb-item v-if="keyword">搜索：“{{ keyword }}”</el-breadcrumb-item>
                            </el-breadcrumb>
                        </div>
                        <span class="total">{{ totalItem }}个项目</span>
                    </div>
                    <el-table
                        :data="filterData"
                        tooltip-effect="dark"
                        style="width: 100%"
                        @selection-change="handleSelectionChange"
                        :default-sort = "{prop: 'time', order: 'descending'}"
                        :empty-text="emptyText"
                        >
                        <el-table-column
                        type="selection"
                        width="55">
                        </el-table-column>
                        <el-table-column
                        prop="name"
                        label="文件"
                        show-overflow-tooltip
                        >
                            <template slot-scope="scope">
                                <span @contextmenu.prevent="rightArrow(scope.$index, $event)" class="ff">
                                    <img src="../../static/img/files.png" alt="" v-if="!scope.row.name.includes('.')" class="folder">
                                    <img src="../../static/img/word.png" alt="" v-else-if="scope.row.name.endsWith('.docx')">
                                    <img src="../../static/img/ppt.png" alt="" v-else-if="scope.row.name.endsWith('.ppt')">
                                    <img src="../../static/img/pdf.png" alt="" v-else-if="scope.row.name.endsWith('.pdf')">
                                    <img src="../../static/img/xlsx.png" alt="" v-else-if="scope.row.name.endsWith('.xlsx')">
                                    {{ scope.row.name }}
                                </span>
                                <div class="more right">
                                    <div class="hide">
                                        <ul>
                                            <li @click="delFile(scope.row.id)">删除</li>
                                            <li @click="resetname(scope.row)">重命名</li>
                                            <li v-show="scope.row.name.includes('.')">
                                                <input type="file">
                                                <span>上传新版本</span>
                                            </li>
                                        </ul>
                                    </div>
                                </div>
                            </template>
                        </el-table-column>
                        <el-table-column
                        label=""
                        width="200"
                        >
                            <template slot-scope="scope">
                                <div class="loadmore">
                                    <a class="download" v-show="scope.row.name.includes('.')" href="" target="_blank">
                                        <img src="../../static/img/download.png" alt="">
                                        下载
                                    </a>
                                    <div class="more">
                                        <div class="show" @click.stop="showOperate(scope.$index, 0)">
                                            更多
                                            <i class="el-icon-arrow-down"></i>
                                        </div>
                                        <div class="hide">
                                            <ul>
                                                <li @click="delFile(scope.row.id)">删除</li>
                                                <li @click="resetname(scope.row)">重命名</li>
                                                <li v-show="scope.row.name.includes('.')" @change="uploadFile($event, scope.row.id)">
                                                    <input type="file" :class="'file' + scope.row.id">
                                                    <span>上传新版本</span>
                                                </li>
                                            </ul>
                                        </div>
                                    </div>
                                </div>
                            </template>
                        </el-table-column>
                        <el-table-column
                        prop="time"
                        label="最新修改"
                        show-overflow-tooltip
                        sortable>
                        </el-table-column>
                    </el-table>
                </div>
            </div>
            <div class="block">
                <el-pagination
                    layout="prev, pager, next"
                    :total="totalItem"
                    :page-size="pageSize"
                    :current-page="currentPage"
                    @current-change="handleCurrentChange">
                </el-pagination>
            </div>
        </div>
        <el-dialog title="新建文件夹" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
            <el-form>
                <el-form-item label="文件夹名称：">
                    <el-input v-model="filename"></el-input>
                </el-form-item>
                <!-- <el-form-item label="所在位置：">
                    <el-select v-model="fileaddr" placeholder="请选择所在位置" :value="labelModel">
                        <el-option :value="valueModel" :label="labelModel">
                            <el-tree :data="filterFiles" :props="defaultProps">
                                <span class="custom-tree-node" slot-scope="{ node }">
                                    {{ node.label }}           
                                </span>
                            </el-tree>
                        </el-option>
                    </el-select>
                </el-form-item> -->
                <div class="btn">
                    <el-button @click="dialogFormVisible = false">取 消</el-button>
                    <el-button type="primary" @click="newConfirm">确 定</el-button>
                </div>
            </el-form>
        </el-dialog>
        <el-dialog title="重命名" :visible.sync="dialogFormVisible2" :close-on-click-modal="false">
            <el-form>
                <el-form-item label="新名称：">
                    <el-input v-model="rname"></el-input>
                </el-form-item>
                <div class="btn">
                    <el-button @click="dialogFormVisible2 = false">取 消</el-button>
                    <el-button type="primary" @click="resetConfirm">确 定</el-button>
                </div>
            </el-form>
        </el-dialog>
    </div>
</template>

<script>
import $ from 'jquery'
import axios from 'axios'
const config = {
    headers: {
        'Content-Type': 'multipart/form-data'
        }
    }
// let tree = [];
// function findFolder(array, arr) {
//     // console.log(arr);
//     let content = [];
//     arr.forEach((item, index) => {
//         if (item.isfolder) {
//             content.push(item);
//             array.children = content;
//             if (item.children) {
//                 findFolder(item, item.children);
//             }
//         }
//     })
// }
export default {
    name: 'database',
    data() {
        return {
            //消息列表
            aMsg: [
                // {
                //     time: '2018-12-31 12:30',
                //     content: '电脑报销制度上传新版本！'
                // },
                // {
                //     time: '2018-12-31 12:30',
                //     content: '电脑报销制度上传新版本！'
                // },
                // {
                //     time: '2018-12-31 12:30',
                //     content: '电脑报销制度上传新版本！'
                // }
            ],
            //常用文件列表
            aFile: [
                {
                    name: "青塔人才介绍.docx",
                    img: ''
                },
                {
                    name: "青塔人才介绍.docx",
                    img: ''
                },
                {
                    name: "青塔人才介绍.docx",
                    img: ''
                }
            ],
            //文件列表
            files: [
                // {   
                //     isfolder: true,
                //     name:"公司资料",
                //     addr:"",
                //     id: "1",
                //     time: '2019-05-03 12:30',
                //     children: [
                //         {
                //             isfolder: true,
                //             addr:"",
                //             id: "2",
                //             name: '销售资料',
                //             time: '2019-05-03 12:30',
                //             children: [
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "323",
                //                     name: '销售资料1.docx',
                //                     time: '2019-05-02 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "13",
                //                     name: '销售资料2.ppt',
                //                     time: '2019-05-03 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "3321",
                //                     name: '销售资料3.pdf',
                //                     time: '2019-05-03 09:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料4.ppt',
                //                     time: '2019-05-01 12:30',
                //                 }
                //             ]
                //         },
                //         {
                //             isfolder: true,
                //             addr:"",
                //             id: "3",
                //             name: '产品资料',
                //             time: '2019-05-03 12:30',
                //             children: [
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "323",
                //                     name: '销售资料.docx',
                //                     time: '2019-05-03 12:30'
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "13",
                //                     name: '销售资料.docx',
                //                     time: '2019-05-03 12:30'
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "3321",
                //                     name: '销售资料.docx',
                //                     time: '2019-05-03 12:30'
                //                 },
                //                 {
                //                     isfolder: false,
                //                     addr:"",
                //                     id: "332",
                //                     name: '销售资料.docx',
                //                     time: '2019-05-03 12:30'
                //                 }
                //             ]
                //         }
                //     ]
                // }
            ],
            //elementui树配置
            defaultProps: {
                children: 'children',
                label: 'name'
            },
            //表格数据
            tableData: [
                // {
                //     index: 0,
                //     id: 0,
                //     name: '公司资料',
                //     time: '2019-05-03 09:30'
                // },
                // {
                //     index: 1,
                //     id: 1,
                //     name: '销售资料',
                //     time: '2019-05-03 12:30'
                // }
            ],
            //分页的表格数据
            filterData: [],
            //搜索关键词
            keyword: "",
            // //是否显示搜索关键词
            // searchShow: false,
            //新建文件夹弹窗是否显示
            dialogFormVisible: false,
            //重命名弹窗是否显示
            dialogFormVisible2: false,
            //新建文件名
            filename: "",
            //重命名文件名称
            rname: "",
            //有后缀的文件名称
            dname: "",
            //重命名文件id
            rid: "",
            //新建文件所在位置
            fileaddr: "",
            //搜索项目总数
            totalItem: 1,
            emptyText: "暂无数据",
            pageNo: 1,
            currentPage: 1,
            pageSize: 10,
            labelModel: "",
            valueModel: "",
            //面包屑导航数组
            crumbArr: [],
            //默认目录显示
            defaultArr: [],
            //文件/文件夹id
            newid: '',
            //批量删除文件/文件夹id
            idarr: []
        }
    },
    methods: {
        //点击跳页
        handleCurrentChange(val){
            var page = val;
            this.pageNo = val;
            this.filterData.splice(0);
            var start = (page-1)*(this.pageSize);
            var end = page*(this.pageSize);
            if(page == Math.ceil(this.totalItem/(this.pageSize))){
                end = this.totalItem;
            }
            for(var i = start; i < end; i ++){
                this.filterData.push(this.tableData[i]);
            }
        },
        //深度克隆对象或数组
        copyObj(obj){
            var str, newobj = obj.constructor === Array ? [] : {};
            if(typeof obj !== 'object'){
                return;
            } else if(window.JSON){
                str = JSON.stringify(obj);//系列化对象
                newobj = JSON.parse(str); //还原
            } else {
                for(var i in obj){
                    newobj[i] = typeof obj[i] === 'object' ? 
                    this.copyObj(obj[i]) : obj[i]; 
                }
            }
            return newobj;
        },
        //左击隐藏右击菜单
        hideUl() {
            this.$nextTick(() => {
                $('.hide ul').hide();
            })
        },
        //右击菜单改变
        rightArrow(val, e) {
            this.showOperate(val, 1);
            this.$nextTick(() => {
                $('.right .hide ul').eq(val).parents('tr').siblings().find('.right .hide ul').hide();
                $('.right .hide ul').eq(val).offset({left: e.clientX, top: e.clientY + $(window).scrollTop()})
            })
        },
        //elementui树结构点击事件
        handleNodeClick(val, node) {
            this.newid = val.id;
            this.keyword = "";
            this.tableData.splice(0);
            this.crumbArr.splice(0);
            this.defaultArr.splice(0);
            this.getData();
            // if (val.children) {
            //     this.tableData = val.children.map(value => {
            //         return {
            //             id: value.id,
            //             name: value.name,
            //             time: value.time,
            //             addr: value.addr,
            //             isfolder: value.isfolder
            //         }
            //     })
            //     this.totalItem = this.tableData.length;
            //     this.beforeData = this.copyObj(this.tableData);
            // } else {
            //     val.index = 0
            //     this.tableData[0] = this.copyObj(val);
            //     this.beforeData[0] = this.copyObj(val);
            //     this.totalItem = 1;
            // }
            let item = node;
            while(item.parent) {
                this.crumbArr.unshift(item.data.name);
                this.defaultArr.unshift(item.data.id);
                item = item.parent;
            }
        },
        //确认是否删除文件
        delFile(val) {
            this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            })
            .then(() => {
                if (val) {
                    this.deleteFile(val);
                } else {
                    this.deleteFile();
                }
            })
            .catch(() => {
                this.$message({
                    type: "info",
                    message: "已取消删除"
                });
            });
        },
        //删除文件
        deleteFile(val) {
            let data = new FormData();
            let arr = [];
            if (val) {
                arr[0] = val;
            } else {
                arr = this.copyObj(this.idarr);
            }
            data.append("idarr", JSON.stringify(arr));
            this.axios.post('api/delete-fd/', data, config).then(res => {
                if (res.data.state == 0) {
                    this.$message({
                        type: "success",
                        message: "删除成功!"
                    });
                    this.getTree();
                    this.getData();
                }
            })
        },
        //批量删除文件
        deleteAll(val) {
            if (val.length > 0) {
                this.delFile();
            } else {
                this.$message({
                    type: "warning",
                    message: "没有选择需要删除的文件/文件夹!"
                });
            }
        },
        //重命名
        resetname(val) {
            this.rname = "";
            this.rid = val.id;
            if (val.name.includes('.')) {
                this.dname = val.name.slice(val.name.indexOf('.'), val.name.length);
            }
            this.dialogFormVisible2 = true;
        },
        //确认重命名
        resetConfirm() {
            let data = new FormData();
            data.append("id", this.rid);
            data.append("name", this.rname + this.dname);
            this.axios.post('api/edit-file/', data, config).then(res => {
                if (res.data.state == 0) {
                    this.$message({
                        type: "success",
                        message: "重命名成功!"
                    });
                    this.dialogFormVisible2 = false;
                    this.getTree();
                    this.getData();
                }
            }) 
        },
        //显示新建文件夹
        showNewFile() {
            if (!this.newid) {
                this.$message({
                    type: "warning",
                    message: "请先选择目录下的文件夹再进行新建文件！"
                });
            } else {
                this.filename = "";
                this.dialogFormVisible = true;
            }
        },
        //确认新建文件夹
        newConfirm() {
            this.dialogFormVisible = false;
            let data = new FormData();
            data.append("id", this.newid);
            data.append("name", this.filename);
            this.axios.post('api/create-folder/', data, config).then(res => {
                if (res.data.state == 0) {
                    this.$message({
                        type: "success",
                        message: "新建文件夹成功!"
                    });
                    this.getTree();
                    this.getData();
                }
            })
        },
        //搜索文件夹
        search() {
            // let that = this;
            // this.tableData = this.copyObj(this.beforeData);
            // this.tableData = this.tableData.filter(val => {
            //     let index = val.name.indexOf('.');
            //     let name = index != -1 ? val.name.slice(0, index) : val.name;
            //     return name.includes(that.keyword);
            // })
            // this.totalItem = this.tableData.length;
            // this.currentPage = 1;
            this.getData(this.keyword);
        },
        //表格选中
        handleSelectionChange(val) {
            this.idarr.splice(0);
            var that = this;
            if (val.length > 0) {
                val.forEach(value => {
                    that.idarr.push(value.id);
                })
            }
        },
        //显示操作
        showOperate(val, isRight) {
            if (isRight) {
                this.$nextTick(() => {
                    $('.right .hide ul').eq(val).show();
                })
            } else {
                this.$nextTick(() => {
                    $('.loadmore .hide ul').eq(val).toggle();
                })
            }
        },
        //上传文件
        uploadFile(e, val) {
            if (!this.newid) {
                this.$message({
                    type: "warning",
                    message: "请先选择目录下的文件夹再进行上传文件！"
                });
            } else {
                let that = this;
                let file = e.target.files;
                let data = new FormData();
                let id = val ? val : this.newid;
                data.append("file", file[0]);
                data.append('id', id);
                that.axios.post('api/upload-file/', data, config).then(res => {
                    if (res.data.state == 0) {
                        that.$message({
                            type: "success",
                            message: "上传文件成功!"
                        });
                        that.getTree();
                        that.getData();
                    }
                })
            }
        },
        //获取文件目录
        getTree() {
            this.axios.post('/api/get-whole-tree/').then(res => {
                if (res.data.state == 0) {
                    this.files = this.copyObj(res.data.data);
                    if (this.crumbArr.length == 0) {
                        this.tableData = this.copyObj(res.data.data);
                        this.filterData = this.copyObj(res.data.data);
                    } else {
                        this.$nextTick(() => {
                            this.$refs.tree.setCurrentKey(this.defaultArr[this.defaultArr.length]);
                        });
                    }
                }
            })
        },
        //获取表格数据
        getData(val) {
            this.tableData.splice(0);
            this.filterData.splice(0);
            this.emptyText = '正在加载中...';
            let data = new FormData();
            if (val) {
                data.append("keyword", val);
            } else {
                data.append("id", this.newid);
            }
            this.axios.post('api/get-file/', data, config).then(res => {
                // console.log(res);
                if (res.data.state == 0) {
                    if (res.data.data.item.length == 0) {
                        this.emptyText = '暂无数据';
                    }
                    this.filterData = this.copyObj(res.data.data.item);
                    this.tableData = this.copyObj(res.data.data.item);
                }
                console.log(this.filterData);
            })
        },
        //获取常用文件
        getCommonFile() {
            this.axios.post('api/get-recent-file/').then(res => {
                if (res.data.state == 0) {
                    // this.aFile = res.data.data;
                }
            })
        },
        //获取最新消息
        getNewMessage() {
            this.axios.post('api/get-message/').then(res => {
                if (res.data.state == 0) {
                    this.aMsg = res.data.data.slice(0, 3);
                }
            })
        },
        //返回上一级
        returnTop() {
            this.newid = this.crumbArr[this.crumbArr.length - 1];
            this.crumbArr.pop();
            this.defaultArr.pop();
            this.getTree();
            this.getData();
        }
    },
    created() {
        this.getNewMessage();
        this.getCommonFile();
        this.getTree();
    }
}
</script>

<style lang="less">
.database {
    overflow: hidden;
    margin-top: 60px;
    min-height: calc(100% - 60px);
    .file-msg {
        float: left;
        padding: 20px;
        width: 250px;
        min-height: calc(100% - 60px);
        border: 1px solid #e4e4e4;
        background-color: #E9EEF1;
        .title {
            margin-bottom: 25px;
            font-size: 14px;
            color: #000;
            img {
                width: 25px;
                height: 25px;
                margin-right: 5px;
                vertical-align: middle;
            }
        }
        ul {
            margin-left: 20px;
            padding-left: 10px; 
            border-left: 1px solid #e4e4e4;
            li {
                margin-bottom: 20px;
                font-size: 13px;
                color: rgba(0, 0, 0, .85);
                a {
                    color: rgba(0, 0, 0, .85);
                }
                img {
                    width: 20px;
                    height: 20px;
                    vertical-align: top;
                    margin-right: 5px;
                }
                span {
                    position: relative;
                    color: rgba(0, 0, 0, .65);
                    &:before {
                        content: '';
                        position: absolute;
                        left: -14px;
                        top: 5px;
                        display: inline-block;
                        width: 8px;
                        height: 8px;
                        border-radius: 50%;
                        background-color: #4CABFF;
                    }
                }
                p {
                    margin-top: 15px;
                    color: #000;
                }
            }
        }
        .file {
            ul {
                li {
                    color: rgba(0, 0, 0, .85);
                }
            }
        }
        .msg {
            padding-top: 20px;
        }
    }
    .files {
        width: 250px;
        min-height: calc(100% - 60px);
        float: left;
        padding: 20px 0;
        border: 1px solid #e4e4e4;
        img {
            width: 20px;
            vertical-align: middle;
            margin-right: 10px;
        }
        .folder {
            width: 25px;
            height: 20px;
        }
        .el-tree-node {
            .el-tree-node__content {
                height: 40px;
                position: relative;
                &:hover {
                    background: rgba(76, 171, 255, .2);
                    &:after {
                        transform: scale(1);
                    }
                }
                &:after {
                    content: '';
                    position: absolute;
                    display: inline-block;
                    right: 0;
                    top: 0;
                    width: 2px;
                    height: 40px;
                    background-color: rgb(76, 171, 255);
                    transform: scale(0);
                    transition: transform .5s;
                }
            }
            .is-current {
                >.el-tree-node__content {
                    background: rgba(76, 171, 255, .2);
                    &:after {
                        transform: scale(1);
                    }
                }
            }
            &:focus {
                >.el-tree-node__content {
                    background: rgba(76, 171, 255, .2);
                    &:after {
                        transform: scale(1);
                    }
                }
            }
        }
    }
    .wrapper {
        .fileshow {
            float: left;
            width: calc(100% - 500px);
            border: 1px solid #e4e4e4;
            border-bottom: none;
            .top {
                padding: 30px 40px;
                overflow: hidden;
                border-bottom: 1px solid #e4e4e4;
                font-size: 14px;
                .upload {
                    position: relative;
                    float: left;
                    width: 100px;
                    height: 35px;
                    cursor: pointer;
                    input {
                        position: absolute;
                        left: 0;
                        top: 0;
                        z-index: 2;
                        width: 90px;
                        opacity: 0;
                        cursor: pointer;
                    }
                    span {
                        position: absolute;
                        left: 0;
                        top: 0;
                        display: inline-block;
                        width: 100px;
                        height: 35px;
                        line-height: 35px;
                        border-radius: 3px;
                        font-size: 14px;
                        text-align: center;
                        color: #fff;
                        background-color: #4CABFF;
                        cursor: pointer;
                    }
                }
                .newfile {
                    float: left;
                    margin-left: 10px;
                    width: 100px;
                    height: 35px;
                    line-height: 35px;
                    border: 1px solid #4CABFF;
                    border-radius: 3px;
                    font-size: 14px;
                    text-align: center;
                    color: #4CABFF;
                    background-color: #fff;
                    cursor: pointer;
                    &:hover {
                        opacity: .7;
                    }
                }
                .el-button--danger {
                    float: left;
                    width: 110px;
                    height: 35px;
                    margin-left: 10px;
                    color: #f56c6c;
                    background-color: #fff;
                    i {
                        display: inline-block;
                        vertical-align: middle;
                        font-size: 18px;
                    }
                    &:hover {
                        opacity: .7;
                    }
                }
                .search {
                    float: right;
                    width: 315px;
                    height: 35px;
                    padding: 0 15px;
                    border-radius: 20px;
                    background-color: #f2f2f2;
                    input {
                        width: 267px;
                        height: 25px;
                        margin-top: 5px;
                        border: none;
                        outline: none;
                        color: rgba(0, 0, 0, .55);
                        background-color: #f2f2f2;
                    }
                    i {
                        cursor: pointer;
                        font-size: 18px;
                    }
                }
            }
            .content {
                .title {
                    overflow: hidden;
                    padding: 30px 40px;
                    border-bottom: 1px solid #e4e4e4;
                    .return {
                        float: left;
                        margin-right: 20px;
                        padding-right: 20px;
                        border-right: 1px solid #e4e4e4;
                        color: #4CABFF;
                    }
                    .breadcrumb {
                        float: left;
                        .is-link {
                            color: #4CABFF;
                            .return {
                                float: left;
                                margin-right: 20px;
                                padding-right: 20px;
                                border-right: 1px solid #e4e4e4;
                                cursor: pointer;
                            }
                        }
                    }
                    .total {
                        float: right;
                        margin-left: 20px;
                        margin-top: 5px;
                    }
                }
                .more {
                    position: relative;
                    float: right;
                    .show {
                        position: relative;
                        z-index: 999;
                        cursor: pointer;
                    }
                    .hide {
                        ul {
                            position: absolute;
                            left: -96px;
                            top: 27px;
                            z-index: 1000;
                            display: none;
                            margin-left: 100px;
                            width: 140px;
                            padding: 20px;
                            border: 1px solid #e4e4e4;
                            text-align: left;
                            background-color: #fff;
                            li {
                                cursor: pointer;
                                color: rgba(0, 0, 0, .85);
                                &:hover {
                                    color: #4CABFF;
                                }
                                &:nth-child(3) {
                                    position: relative;
                                    input {
                                        position: absolute;
                                        left: 0;
                                        top: 0;
                                        z-index: 2;
                                        width: 90px;
                                        opacity: 0;
                                        cursor: pointer;
                                    }
                                }
                            }
                        }
                    }
                }
                .loadmore {
                    user-select: none;
                    // overflow: hidden;
                    >div {
                        display: inline-block;
                        color: #4CABFF;
                    }
                    .download {
                        float: left;
                        display: inline-block;
                        color: #4CABFF;
                        cursor: pointer;
                        img {
                            width: 16px;
                            height: 16px;
                        }
                    }
                    
                }
            }
            .el-table {
                overflow: visible;
                .el-table__body-wrapper {
                    overflow: visible;
                }
                .cell {
                    overflow: visible;
                    text-align: center
                }
                thead {
                    th {
                        font-weight: lighter;
                        color: #000;
                    }
                    background-color: #f8f8f8; 
                }
                tr {
                    color: rgba(0, 0, 0, .85);
                }
                .ff {
                    img {
                        width: 20px;
                        vertical-align: middle;
                        margin-right: 10px;
                    }
                    .folder {
                        width: 25px;
                        height: 20px;
                    }
                }
            }
        }
        .block {
            .el-pagination {
                padding: 30px 20px;
                float: right;
            }
        }
    }
    .btn {
        text-align: right;
    }
    .el-dialog__header {
        padding: 20px;
        border-bottom: 1px solid #dcdfe6;
    }
    .el-select {
        width: 100%;
    }
    .custom-tree-node > span {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
}
.el-select-dropdown__item {
    height: auto !important;
    line-height: auto !important;
    overflow: visible !important;
}
.el-tree-node__content {
    height: auto;
}
</style>
