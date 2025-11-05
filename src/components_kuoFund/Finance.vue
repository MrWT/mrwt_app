<script setup>
    import { ref, reactive, onMounted, watch } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        user_role: String,
    })

    onMounted(() => {
        console.log("Finance mounted.");
        init();
    });

    let userRole = ref("");
    let appState = ref("");

    let funds_total = ref(0);
    let funds_members = reactive([]);
    let funds = reactive([]);
    let funds_months = reactive([]);
    let sel_dataMN = ref("");

    let delRecordObj = reactive({});

    // 初始化 component
    function init(){
        console.log("Finance.init");
        console.log("Finance.props.title", props.title);
        console.log("Finance.props.account", props.account);
        console.log("Finance.props.user_role", props.user_role);
        userRole.value = props.user_role;
        
        // 取得初始資料
        fetchInitData();
    }    
    // 取得初始資料
    function fetchInitData(){
        //console.log("fetchInitData");
        // 清空結餘
        funds_total.value = 0;
        // 取得資料月份
        let fetchFundsMonthPromise = fetchData({
            api: "get_finance_month",
        }, "KUO-FUNDS");
        // 取得總結餘
        let fetchFundsTotalPromise = fetchData({
            api: "get_finance_total",
        }, "KUO-FUNDS");
        // 取得成員
        let fetchMembersPromise = fetchData({
            api: "get_members",
        }, "KUO-FUNDS");
        Promise.all([fetchFundsTotalPromise, fetchFundsMonthPromise, fetchMembersPromise]).then((values) => {
            console.log("fetchFundsPromise.values=", values);

            funds_total.value = values[0];

            // clear month list
            funds_months.splice(0, funds_months.length);
            values[1].forEach((fmn, fmn_i) => {
                funds_months.push(fmn);
            });

            if(!funds_months.includes(moment().format("YYYY-MM"))){
                funds_months.push(moment().format("YYYY-MM"));
            }

            funds_months.sort((x, y) => {
                if( x < y ){
                    return 1;
                }
                if( x > y ){
                    return -1;
                }
                return 0;
            });

            // clear member list
            funds_members.splice(0, funds_members.length);
            values[2].forEach((fmn, fmn_i) => {
                funds_members.push(fmn);
            });
        }).then(() => {
            sel_dataMN.value = moment().format("YYYY-MM");
        });
    }
    // 選擇查閱的資料月份
    function clickDataMN(dataYM){
        sel_dataMN.value = dataYM;
    }
    // 取得特定月份儲值/提領紀錄
    function fetchFunds(dataYM){
        console.log("fetchFunds.dataYM=", dataYM);

        let fetchFundsPromise = fetchData({
            api: "get_finance",
            data: {
                dataYM: dataYM,
            },
        }, "KUO-FUNDS");
        Promise.all([fetchFundsPromise]).then((values) => {
            console.log("fetchFundsPromise.values=", values);

            funds.splice(0, funds.length);
            values[0].forEach((fundObj, f_i) => {
                // 找到紀錄姓名
                for(let mem_i = 0; mem_i < funds_members.length; mem_i++){
                    let memObj = funds_members[mem_i];
                    if(memObj["code_name"] === fundObj["code_name"]){
                        fundObj["name"] = memObj["name"];
                        break;
                    }
                }
                funds.push(fundObj);
            });
            // 排序
            funds.sort((x, y) => {
                // 依 code_name 排序
                if(x["code_name"] > y["code_name"]) return 1;
                if(x["code_name"] < y["code_name"]) return -1;
                if(x["code_name"] === y["code_name"]){
                    // 依日期排序
                    if( x["date"] < y["date"] ) return 1;
                    if( x["date"] > y["date"] ) return -1;
                    if( x["date"] === y["date"] ) return 0;
                }
            });
        });

    }
    // 跳出失效再確認 modal
    function popupDelConfirmModal(delObj){
        delRecordObj["code_name"] = delObj["code_name"];
        delRecordObj["name"] = delObj["name"];
        delRecordObj["type"] = delObj["type"];
        delRecordObj["date"] = delObj["date"];
        console.log("popupDelConfirmModal.delRecordObj=", delRecordObj);

        document.getElementById("delConfirmModal").showModal();
    }
    // 取消失效再確認 modal
    function cancelPopupDelConfirmModal(){
        document.getElementById("delConfirmModal").close();
    }
    // 失效儲值/提領紀錄
    function deleteRecord(){
        console.log("deleteRecord.delRecordObj=", delRecordObj);

        let delFinanceKFPromise = fetchData({
            api: "del_finance",
            data: {
                finance: delRecordObj,
            }
        }, "KUO-FUNDS");
        Promise.all([delFinanceKFPromise]).then((values) => {
            console.log("delFinanceKFPromise.values=", values);

            let opObj = {
                message: "",
                status: true,
            };
            opObj.status = values[0]["result"];
            if(values[0]["result"] === true){
                opObj.message = "失效成功";
                fetchInitData();
            }else{
                opObj.message = values[0]["message"];
            }

            document.getElementById("delConfirmModal").close();
            // 將 message 傳給 Setting.vue 
            emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data
        });
    }

    // 監聽 sel_dataMN
    watch(sel_dataMN, (newDataMN, oldDataMN) => {
        console.log("watch.sel_dataMN.newDataMN=", newDataMN);

        fetchFunds(newDataMN);
    });
</script>

<template>

<div class="w-1/1 overflow-x-auto overflow-y-hidden shadow-2xl">
    <button v-for="(f_mn, f_mn_i) in funds_months" @click="clickDataMN(f_mn)"
                class="btn btn-ghost rounded-none border-0 border-b-2 hover:border-rose-500 mx-1"
                :class="{'border-amber-600': f_mn === sel_dataMN, 'border-gray-500': f_mn !== sel_dataMN}">
        {{ f_mn }}
    </button>
</div>
<div class="w-1/1 h-4/6 mt-2 flex flex-col overflow-y-auto">
    <div v-for="(fundObj, fund_i) in funds" class="chat"
        :class="{ 'chat-start': fundObj.type === 'IN', 'chat-end': fundObj.type === 'OUT' }">
        <div class="chat-image avatar">
            <div class="avatar avatar-placeholder">
                <div class="w-12 rounded-full"
                    :class="{'bg-green-900': fundObj.type === 'IN', 'bg-red-900': fundObj.type === 'OUT'}">
                    <span v-if="fundObj.type === 'IN'" class="text-lg text-white">儲值</span>
                    <span v-if="fundObj.type === 'OUT'" class="text-lg text-white">提領</span>
                </div>
            </div>
        </div>
        <div class="chat-header">
            <span class="text-base">{{ fundObj.name }}</span>
            <time class="text-base opacity-50">{{ fundObj.date }}</time>
        </div>
        <div class="chat-bubble">
            <span v-if="userRole === 'admin_kf'" class="mr-2 font-black text-red-900 cursor-pointer" @click="popupDelConfirmModal(fundObj)">X</span>
            $ {{ new Intl.NumberFormat().format( fundObj.money ) }}
            <span v-if="fundObj.memo !== ''">( {{ fundObj.memo }} )</span>
        </div>
    </div>
    <div v-if="funds.length === 0" class="text-3xl text-center w-1/1">查無資料</div>
</div>

<div class="w-10/10 h-1/6 mt-1 flex flex-col text-3xl justify-center items-center rounded-full shadow-xl" :class="{'bg-gray-200': funds_total === 0, 'bg-green-200': funds_total > 0, 'bg-red-200': funds_total < 0}">
    結餘：$ {{ new Intl.NumberFormat().format( funds_total ) }}
</div>

<!-- 失效再確認 -->
<dialog id="delConfirmModal" class="modal">
    <div class="modal-box bg-gray-800/80 rounded-box p-2 w-80 md:w-120">
        <li class="text-white text-xl">{{ "確定要失效以下資料嗎?" }}</li>
        <div class="divider divider-primary"></div>
        <li class="text-white text-lg">姓名: {{ delRecordObj.name }}</li>
        <li class="text-white text-lg">儲值或提領: {{ delRecordObj.type === "IN" ? "儲值" : "提領" }}</li>
        <li class="text-white text-lg">日期: {{ delRecordObj.date }}</li>
        <div class="divider divider-primary"></div>
        <li class="flex flex-row gap-2 w-10/10">
            <button class="btn w-5/10" @click="cancelPopupDelConfirmModal">
                取消
            </button>
            <button class="btn w-5/10" @click="deleteRecord">
                確認
            </button>
        </li>
    </div>
    <form method="dialog" class="modal-backdrop">
        <button>close</button>
    </form>
</dialog>

</template>

<style scoped>

</style>
