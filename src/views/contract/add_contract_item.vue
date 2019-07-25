<template>
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true">
                <el-form-item>
                    <el-button type="primary" @click="addContractItem('itemFrom')">保存条款</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="cancelContractItem">取  消</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <el-form :model="itemFrom" :label-position="labelPosition" @submit.native.prevent  label-width="110px" style="margin:20px;width:90%;min-width:600px;">
            <el-form-item label="条款编号">
                <el-input v-model="itemFrom.termId" placeholder="请选择条款编号" style="width: 200px"></el-input>
            </el-form-item>
            <el-row>
                <el-col :span="6">
                    <el-form-item label="物流服务项">
                        <!--   auto-complete="off"禁止表格自动填充   readonly禁止表格编辑  -->
                        <el-input v-model="itemFrom.logisticsServiceItem.name" readonly placeholder="请选择物流服务项" style="width: 90%"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="1">
                    <el-button type="primary" @click="toSelectLogisticsServiceItem">选择物流服务项</el-button>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="6">
                    <el-form-item label="物流服务项类型">
                        <el-input title="自动填充不可编辑" v-model="itemFrom.logisticsServiceItem.type" readonly style="width: 90%"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item label="位置（起）">
                        <el-input title="自动填充不可编辑" v-model="itemFrom.logisticsServiceItem.startAddress" readonly style="width: 90%"></el-input>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item label="位置（止）">
                        <el-input title="自动填充不可编辑" v-model="itemFrom.logisticsServiceItem.endAddress" readonly style="width: 90%"></el-input>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="6">
                    <el-form-item label="服务项变相">
                        <el-select v-model="itemFrom.addition" placeholder="请选择变相">
                            <el-option label="无变相" value="0"></el-option>
                            <el-option label="刷卡支付" value="1"></el-option>
                            <el-option label="现金支付" value="2"></el-option>
                        </el-select>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item label="启用团购计数" title="（规定同一“出库日期”、和“起”“止”地汇相同的为团购）">
                        <el-switch on-text="是" off-text="否" v-model="itemFrom.group"></el-switch>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="6">
                    <el-form-item label="取值精度">
                        <el-select v-model="itemFrom.numericalAccuracy" placeholder="请选择取值精度">
                            <el-option label="1" value="1"></el-option>
                            <el-option label="2" value="2"></el-option>
                            <el-option label="3" value="3"></el-option>
                        </el-select>
                    </el-form-item>
                </el-col>
                <el-col :span="6">
                    <el-form-item label="计算结果精度">
                        <el-select v-model="itemFrom.resultAccuracy" placeholder="请选择取值精度">
                            <el-option label="0" value="0"></el-option>
                            <el-option label="1" value="1"></el-option>
                            <el-option label="2" value="2"></el-option>
                            <el-option label="3" value="3"></el-option>
                        </el-select>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-form-item label="舍入规则" title="取值精度的舍入规则">
                <el-radio-group v-model="itemFrom.roundoffRule">
                    <el-radio label="1">四舍五入（2.1记为2，2.6记为3）</el-radio>
                    <el-radio label="2">0.5进位计整（2.1记为2.5，2.6记为3）</el-radio>
                    <el-radio label="3">进一计整（2.1记为3）</el-radio>
                </el-radio-group>
            </el-form-item>
            <el-form-item label="价格">
                <el-radio-group v-model="itemFrom.priceType" @change="priceTypeFilter">
                    <el-radio label="1">固定价格</el-radio>
                    <el-radio label="2">价格表</el-radio>
                </el-radio-group>
            </el-form-item>
            <el-form-item label="固定价格（￥）" :hidden="single">
                <el-input title="单位为元" v-model="itemFrom.singlePrice" style="width: 90%"></el-input>
            </el-form-item>
            <el-form-item label="价格表" :hidden="priceTableType">
                <el-table :data="priceTable" style="width: 70%" border>
                    <el-table-column header-align="center" label="条件与计算方法" >
                        <el-table-column header-align="center" prop="condition" label="计算条件" ></el-table-column>
                        <el-table-column header-align="center" label="操作" width="200">
                            <template slot-scope="scope">
                                <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                                <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">作废</el-button>
                            </template>
                        </el-table-column>
                    </el-table-column>
                </el-table>
            </el-form-item>
            <el-form-item :hidden="priceTableType">
                <el-button type="primary" @click="toAddRule">新增条件</el-button>
            </el-form-item>
        </el-form>
        <!--   物流服务项选择     -->
        <el-dialog title="物流服务项选择" v-model="selectlogisticsServiceItem" :close-on-click-modal="false" style="width: 100%">
            <!--工具条-->
            <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
                <el-form :inline="true" :model="filters">
                    <el-form-item label="物流服务类型：">
                        <el-select v-model="filters.type" placeholder="请选择物流服务类型查找">
                            <el-option label="干线" value="1"></el-option>
                            <el-option label="宅配正向" value="2"></el-option>
                            <el-option label="宅配拒退换" value="3"></el-option>
                            <el-option label="妥投保价费" value="4"></el-option>
                            <el-option label="拒收保价费" value="5"></el-option>
                            <el-option label="退货保价费" value="6"></el-option>
                            <el-option label="金流费_货到刷卡" value="7"></el-option>
                            <el-option label="金流费_货到付现" value="8"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" v-on:click="getLogisticsServiceItem('filters')">搜索服务项</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-table :data="logisticsServiceItem" highlight-current-row  style="width: 100%;">
                <el-table-column type="index">
                </el-table-column>
                <el-table-column prop="itemId" label="物流服务项ID" >
                </el-table-column>
                <el-table-column prop="name" label="物流服务项名称">
                </el-table-column>
                <el-table-column prop="startAddress" label="位置（起）">
                </el-table-column>
                <el-table-column prop="endAddress" label="位置（止）">
                </el-table-column>
                <el-table-column prop="type" label="物流服务项类型">
                </el-table-column>
                <el-table-column label="操作">
                    <template scope="scope">
                        <el-button size="small" @click="handleChoose(scope.$index, scope.row)">选择</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-dialog>

        <el-dialog title="维护计价规则" v-model="valuationRules" :close-on-click-modal="false" style="width: 100%">
            <!--工具条-->

                    <el-form :model="addRule" label-width="80px" :rules="addRuleRules">
                        <el-row>
                            <el-col :span="5">
                                <el-form-item label="如果：" style="width: 200px">
                                    <el-select v-model="addRule.dimensionality" placeholder="计费维度">
                                        <el-option label="数量" value="1"></el-option>
                                        <el-option label="重量" value="2"></el-option>
                                        <el-option label="体积" value="3"></el-option>
                                        <el-option label="金额" value="4"></el-option>
                                    </el-select>
                                </el-form-item>
                            </el-col>
                            <el-col :span="5">
                                <el-form-item label="" style="width: 200px">
                                    <el-select v-model="addRule.symbol1">
                                        <el-option label=">" value="1"></el-option>
                                        <el-option label=">=" value="2"></el-option>
                                        <el-option label="<" value="3"></el-option>
                                        <el-option label="<=" value="4"></el-option>
                                    </el-select>
                                </el-form-item>
                            </el-col>
                            <el-col :span="5">
                                <el-form-item label="" style="width: 150px" :title="messages">
                                    <el-input auto-complete="off" v-model="addRule.symbolValue1"></el-input>
                                </el-form-item>
                            </el-col>
                        </el-row>
                        <el-row>
                            <el-col :span="5">
                                <el-form-item label="" style="width: 200px">
                                </el-form-item>
                            </el-col>
                            <el-col :span="5">
                                <el-form-item label="" style="width: 200px">
                                    <el-select v-model="addRule.symbol2">
                                        <el-option label=">" value="1"></el-option>
                                        <el-option label=">=" value="2"></el-option>
                                        <el-option label="<" value="3"></el-option>
                                        <el-option label="<=" value="4"></el-option>
                                    </el-select>
                                </el-form-item>
                            </el-col>
                            <el-col :span="5">
                                <el-form-item label="" style="width: 150px" :title="messages">
                                    <el-input auto-complete="off" v-model="addRule.symbolValue2"></el-input>
                                </el-form-item>
                            </el-col>
                        </el-row>
                        <el-row>
                            <el-col :span="5">
                                <el-form-item label="首数：" style="width: 200px">
                                    <el-input auto-complete="off" v-model="addRule.firstQuantity"></el-input>
                                </el-form-item>
                            </el-col>
                        </el-row>
                        <el-row>
                            <el-col :span="4">
                                <el-form-item label="价格 =" style="width: 200px">
                                    <el-input auto-complete="off" v-model="addRule.fixedPrice"></el-input>
                                </el-form-item>
                            </el-col>
                            <el-col :span="1">
                                <el-form-item label="+" style="width: 50px">
                                </el-form-item>
                            </el-col>
                            <el-col :span="4">
                                <el-form-item label="" style="width: 200px">
                                    <el-input auto-complete="off" v-model="addRule.singlePrice"></el-input>
                                </el-form-item>
                            </el-col>
                            <el-col :span="1">
                                <el-form-item label="X" style="width: 50px">
                                </el-form-item>
                            </el-col>
                            <el-col :span="8">
                                <el-form-item label="" style="width: 200px">
                                    <el-select v-model="addRule.dimensionality" placeholder="计费维度">
                                        <el-option label="数量" value="1"></el-option>
                                        <el-option label="重量" value="2"></el-option>
                                        <el-option label="体积" value="3"></el-option>
                                        <el-option label="金额" value="4"></el-option>
                                    </el-select>
                                </el-form-item>
                            </el-col>
                            <el-col :span="4">
                                <el-select v-model="addRule.subFirstQuantity">
                                    <el-option label="减首数" value="1"></el-option>
                                    <el-option label="N/A" value="2"></el-option>
                                </el-select>
                            </el-col>
                        </el-row>
                        <el-row style="left: 80px">
                            <el-col :span="5">
                                <el-button size="big" type="primary" @click="saveRuleAndClose('addRule')">保存并关闭</el-button>
                            </el-col>
                            <el-col :span="5">
                                <el-button size="big" type="primary" @click="saveRuleAndNew('addRule')">保存并新建</el-button>
                            </el-col>
                            <el-col :span="5">
                                <el-button size="big" type="primary" @click="cancel">取 消</el-button>
                            </el-col>
                        </el-row>
                        <el-row>
                            <el-col :span="8">
                                <h3 style="color: #4db3ff">本计费维度已包含规则：</h3>
                            </el-col>
                        </el-row>
                        <el-form-item>
                            <el-table :data="priceTable" style="width: 90%" border>
                                <el-table-column header-align="center" label="条件与计算方法" >
                                    <el-table-column header-align="center" prop="condition" label="计算条件" ></el-table-column>
                                    <el-table-column header-align="center" label="操作" width="150px">
                                        <template slot-scope="scope">
                                            <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                                            <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">作废</el-button>
                                        </template>
                                    </el-table-column>
                                </el-table-column>
                            </el-table>
                        </el-form-item>
                    </el-form>
<!--                    <div slot="footer" class="dialog-footer">-->
<!--                        <el-button @click.native="addFormVisible = false">取消</el-button>-->
<!--                        <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>-->
<!--                    </div>-->
        </el-dialog>
    </section>
</template>

<script>
    export default {
        data(){
            return{
                labelPosition:'left',   //表单对齐样式（默认：right  还有top）
                itemFrom:{
                    termId:'',
                    logisticsServiceItem:{
                        id:'',
                        name:'',
                        type:'',
                        startAddress:'',
                        endAddress:''
                    },
                    addition:'0',
                    group:'否',
                    numericalAccuracy:'1',
                    resultAccuracy:'2',
                    roundoffRule:'1',
                    priceType:'',
                    singlePrice:''
                },
                filters:{
                    type:'',
                },
                single:true,
                priceTableType:true,
                selectlogisticsServiceItem:false,
                priceTable:[{
                    condition:"如果：商品金额 <= 500元，价格 = 20 + 0.01 * 商品金额"
                },{
                    condition:"如果：商品体积 <= 50，价格 = 10 + 0.2 * 商品体积"
                },{
                    condition:"如果：商品重量 <= 5KG，价格 = 15 + 1.00 * 商品重量"
                }],
                logisticsServiceItem:[{
                    itemId:'A01',
                    name:'渝广线干线',
                    startAddress:'重庆',
                    endAddress:'广州',
                    type:'干线'
                },{
                    itemId:'B02',
                    name:'渝广线汽运',
                    startAddress:'重庆',
                    endAddress:'广州',
                    type:'宅配正向'
                },{
                    itemId:'S002',
                    name:'南京刷卡金流费',
                    startAddress:'南京',
                    endAddress:'南京',
                    type:'金流费'
                },{
                    itemId:'S003',
                    name:'上海现金金流费',
                    startAddress:'上海',
                    endAddress:'上海',
                    type:'金流费'
                }],
                valuationRules:true,
                addRule:{
                    dimensionality:'',
                    symbol1:'',
                    symbolValue1:'',
                    symbol2:'',
                    symbolValue2:'',
                    firstQuantity:'',
                    fixedPrice:'',
                    singlePrice:'',
                    subFirstQuantity:'1'
                },
                messages:'',
            }
        },
        methods: {
            //添加合同条款
            addContractItem(form){
                alert("已添加合同条款");
            },
            //取消添加合同
            cancelContractItem:function () {
                this.$confirm('确认放弃添加(或修改)合同条款吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$router.replace('/add_contract')
                }).catch(() => {

                });
            },
            //选择物流服务项
            toSelectLogisticsServiceItem:function(){
                this.selectlogisticsServiceItem = true;
            },
            //价格显示
            priceTypeFilter:function () {
                if (this.itemFrom.priceType === "1"){
                    this.priceTableType = true;
                    this.single = false;
                }else if(this.itemFrom.priceType === "2"){
                    this.single  = true;
                    this.priceTableType = false;
                }
            },
            //添加计价规则
            toAddRule:function () {
                this.valuationRules=true;
            },
            //数据渲染
            handleChoose:function (index, row) {
                this.itemFrom.logisticsServiceItem.name = row.name;
                this.itemFrom.logisticsServiceItem.type = row.type;
                this.itemFrom.logisticsServiceItem.startAddress = row.startAddress;
                this.itemFrom.logisticsServiceItem.endAddress = row.endAddress;
                this.selectlogisticsServiceItem = false;
            },
            //查找物流服务项
            getLogisticsServiceItem(filters) {
                alert("查找物流服务项");
                //alert(filters.type);
            },
            //保存规则并关闭
            saveRuleAndClose(rule){

            },
            //保存规则并新建
            saveRuleAndNew(rule){

            },
            //取消保存
            cancel:function () {

            }
            // chooseDimensionality:function () {
            //     var a = this.addRule.dimensionality;
            //     alert(a);
            //     if (a===1){
            //         this.messages = "单位为kg（示例：2.05）";
            //     } else if (a===2){
            //
            //     } else if (a===3){
            //
            //     }else if (a===4){
            //
            //     }
            // }
        }
    }
</script>

<style scoped>

</style>
