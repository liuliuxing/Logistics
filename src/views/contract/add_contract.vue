<template>
    <el-form ref="form" :model="contractForm" label-width="90px" :rules="addContractRules" @submit.prevent="onSubmit" style="margin:20px;width:90%;min-width:600px;">
        <el-form-item>
            <el-button type="primary" @click="addContract('contractFrom')">保存</el-button>
            <el-button type="primary" @click="cancelContract">取消</el-button>
        </el-form-item>
        <el-row>
            <el-col :span="7">
                <el-form-item label="合作方">
                    <!--   auto-complete="off"禁止表格自动填充   readonly禁止表格编辑  -->
                    <el-input v-model="contractForm.partnerId" readonly placeholder="请选择合作方" style="width: 90%"></el-input>
                </el-form-item>
            </el-col>
            <el-col :span="10">
                <el-button type="primary" @click="toSelectPartner">选择承运人</el-button>
            </el-col>
        </el-row>
        <el-form-item label="合同名称">
            <!--   auto-complete="off"禁止表格自动填充      -->
            <el-input v-model="contractForm.name" contenteditable="true" placeholder="请输入合同名称" auto-complete="off" style="width: 50%"></el-input>
        </el-form-item>
        <el-row>
            <el-col :span="6">
                <el-form-item label="合同编号">
                    <el-input v-model="contractForm.num" contenteditable="true" placeholder="请输入合同编号" auto-complete="off" style="width: 110%"></el-input>
                </el-form-item>
            </el-col>
            <!-- 占位  -->
            <el-col :span="1">
                <span >&nbsp;</span>
            </el-col>
            <el-col :span="6">
                <el-form-item label="合同版本号">
                    <el-input v-model="contractForm.version" contenteditable="true" placeholder="请输入合同版本号" style="width: 110%;"></el-input>
                </el-form-item>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="6">
                    <el-form-item label="生效日期">
                    <el-date-picker type="date" placeholder="选择合同生效日期" v-model="contractForm.startDate" style="width: 110%;"></el-date-picker>
                </el-form-item>
            </el-col>
            <!-- 占位  -->
            <el-col :span="1">
                <span >&nbsp;</span>
            </el-col>
            <el-col :span="6">
                <el-form-item label="截止日期">
                    <el-date-picker type="date" placeholder="选择合同截止日期" v-model="contractForm.endDate" style="width: 110%;"></el-date-picker>
                </el-form-item>
            </el-col>
        </el-row>
        <el-form-item label="立即生效" :title="message">
            <el-switch on-text="1" off-text="0" v-model="contractForm.delivery"></el-switch>
        </el-form-item>


        <el-row>
            <el-col :span="4">
                <h3 style="color: #4db3ff">本合同已包含条款：</h3>
            </el-col>
        </el-row>
        <el-table :data="contractTerms" highlight-current-row  style="width: 100%;">
            <el-table-column type="index" width="55">
            </el-table-column>
<!--            <el-table-column prop="id" :show-overflow-tooltip="false" width="0">-->
<!--            </el-table-column>-->
            <el-table-column prop="termId" label="条款编号" width="120" sortable>
            </el-table-column>
            <el-table-column prop="logisticsServiceItem" label="物流服务项" width="125" sortable>
            </el-table-column>
            <el-table-column prop="type" label="物流服务类型" width="140" sortable>
            </el-table-column>
            <el-table-column prop="startAddress" label="位置（起）" width="150" sortable>
            </el-table-column>
            <el-table-column prop="endAddress" label="位置（止）" min-width="150" sortable>
            </el-table-column>
            <el-table-column label="操作" width="180">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <div slot="footer" class="dialog-footer">
            <el-button @click.native="addFormVisible = false">取消</el-button>
            <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
        </div>
        <el-dialog title="承运商选择" v-model="selectPartner" :close-on-click-modal="false" style="width: 100%">
            <el-table :data="partner" highlight-current-row  style="width: 100%;">
                <el-table-column type="index">
                </el-table-column>
                <!--            <el-table-column prop="id" :show-overflow-tooltip="false" width="0">-->
                <!--            </el-table-column>-->
                <el-table-column prop="id" label="承运商编码" >
                </el-table-column>
                <el-table-column prop="name" label="承运商名称">
                </el-table-column>
                <el-table-column label="操作">
                    <template scope="scope">
                        <el-button size="small" @click="handleChoose(scope.$index, scope.row)">选择</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-dialog>
        <el-row style="top: 7px;">
            <el-col :span="3">
                <el-button type="primary" @click="AddContractTerm">新建合同条款</el-button>
            </el-col>
        </el-row>
    </el-form>
</template>

<script>
    export default {
        data() {
            return {
                contractForm: {
                    partnerId:'',    //合作方
                    name: '',       //合同名称
                    num:'',         //合同编号
                    version:'',     //版本号
                    startDate: '',
                    endDate: '',
                    delivery: false  //立即生效
                },
                message:"每个承运商只能有一个当下生效的合同",
                contractTerms:[{
                    termId: "SF001",
                    logisticsServiceItem: "渝广线干线",
                    type: "干线",
                    startAddress: "重庆",
                    endAddress: "广州"
                }, {
                    termId: "YD002",
                    logisticsServiceItem: "南京宅配逆向",
                    type: "宅配拒退换",
                    startAddress: "南京",
                    endAddress: "南京"
                },{
                    termId: "YD002",
                    logisticsServiceItem: "上海宅配到家",
                    type: "宅配可以退换货物",
                    startAddress: "上海虹桥机场中转站",
                    endAddress: "上海配送基地"
                }],
                addContractRules: {
                    name: [
                        { required: true, message: '请输入合同名字', trigger: 'blur' }
                    ],
                    num: [
                        { required: true, message: '请输入合同编号', trigger: 'blur' }
                    ],
                    startDate: [
                        { required: true, message: '请输入合同生效日期', trigger: 'blur' }
                    ],
                    endDate: [
                        { required: true, message: '请输入合同结束日期', trigger: 'blur' }
                    ]
                },
                selectPartner:false,    //选择承运商页面是否显示
                partner:[{
                    id: "SF",
                    name: "顺丰"
                },{
                    id: "ST",
                    name: "申通"
                },{
                    id: "YD",
                    name: "韵达"
                }]
            };
        },
        methods: {
            addContract(formName){
                this.$refs[formName].validate((valid) =>{
                    if(valid){
                        alert(valid);
                    }else {
                        alert("哇咔咔");
                        return false;
                    }
                })
            },
            //显示选择运营商页面
            toSelectPartner:function () {
                this.selectPartner = true;
            },
            handleChoose:function (index,row) {
                this.contractForm.partnerId = row.name;
                this.selectPartner = false;
            },
            AddContractTerm:function () {
                this.$router.replace('/add_contract_item')
            },
            cancelContract:function () {
                this.$confirm('确认放弃添加(或修改)合同吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$router.replace('/contract')
                }).catch(() => {

                });
            },
            getParams(){
                //this.index = this.$router.query.index;
                this.row = this.$router.query.row;
                this.contractForm.partnerId = this.row.partner;  //合作方
                this.contractForm.name= this.row.contractName;   //合同名称
                this.contractForm.num= this.row.contractNum;   //合同编号
                this.contractForm.version= this.row.versionNum;   //合同版本号
                this.contractForm.startDate= this.row.startDate;   //合同版本号
                this.contractForm.endDate= this.row.endDate;   //合同版本号
                this.contractForm.delivery= this.row.status ==='1'? true:false;   //合同版本号
            }
        }
    }

</script>
