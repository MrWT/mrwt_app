<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        user_role: String,
    })

    onMounted(() => {
        console.log("Finance_KF mounted.");
        init();
    });

    let userRole = ref("");
    let appState = ref("");
    let total_fund = ref(0);
    let funds = reactive({});
    let fundKeys = reactive([]);
    let delRecordObj = reactive({});

    // 初始化 component
    function init(){
        console.log("Finance_KF.init");
        console.log("Finance_KF.props.title", props.title);
        console.log("Finance_KF.props.account", props.account);
        console.log("Finance_KF.props.user_role", props.user_role);
        userRole.value = props.user_role;
        
        // 取得儲值/提領紀錄
        fetchFunds();
    }    
    // 取得儲值/提領紀錄
    function fetchFunds(){
        //console.log("fetchFunds");
        // 清空結餘
        total_fund.value = 0;

        let fetchFundsPromise = fetchData({
            api: "get_kuo_funds",
        });
        let fetchMembersPromise = fetchData({
            api: "get_members_kf",
        });
        Promise.all([fetchFundsPromise, fetchMembersPromise]).then((values) => {
            console.log("fetchFundsPromise.values=", values);
            let members = values[1];

            // 清空 funds
            Object.keys(funds).forEach(key => delete funds[key]);
            values[0].forEach((fundObj, fund_i) => {
                let fund_date = moment(fundObj["date"]).format("YYYYMM");
                if(Object.keys(funds).indexOf(fund_date) < 0){
                    funds[fund_date] = [];
                }

                // 找到紀錄姓名
                for(let mem_i = 0; mem_i < members.length; mem_i++){
                    let memObj = members[mem_i];
                    if(memObj["code_name"] === fundObj["code_name"]){
                        fundObj["name"] = memObj["name"];
                        break;
                    }
                }

                // 依月份建立帳務紀錄
                funds[fund_date].push( fundObj );
                // 順道計算結餘
                if(fundObj["type"] === "IN"){
                    total_fund.value += fundObj["money"];
                }else{
                    total_fund.value -= fundObj["money"];
                }
            });

            Object.keys(funds).forEach((fundKey, fk_i) => {
                // 各個月份之中的帳務紀錄, 各自排序
                funds[fundKey].sort((x, y) => {
                    if(x["code_name"] > y["code_name"]) return 1;
                    if(x["code_name"] < y["code_name"]) return -1;
                    if(x["code_name"] === y["code_name"]) return 0;
                });
            });

            // 排(倒)序月份
            fundKeys = Object.keys(funds);
            fundKeys.sort((x, y) => {
                return parseInt(y) - parseInt(x);
            });

            console.log("funds=", funds);
            console.log("total_fund=", total_fund.value);
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
            api: "del_kuo_funds",
            data: {
                finance: delRecordObj,
            }
        });
        Promise.all([delFinanceKFPromise]).then((values) => {
            console.log("delFinanceKFPromise.values=", values);

            let opObj = {
                message: "",
                status: true,
            };
            opObj.status = values[0]["result"];
            if(values[0]["result"] === true){
                opObj.message = "失效成功";
                fetchFunds();
            }else{
                opObj.message = values[0]["message"];
            }

            document.getElementById("delConfirmModal").close();
            // 將 message 傳給 Setting.vue 
            emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data
        });
    }
</script>

<template>

<div class="w-10/10 md:w-6/10 h-8/10 md:h-8/10 justify-self-center">
    <div class="tabs tabs-border w-10/10 h-10/10">
        <template v-for="(fundKey, fk_i) in fundKeys">
            <input type="radio" name="funds_tabs" class="tab" :aria-label="fundKey.substr(0, 4) + '-' + fundKey.substr(4)" :checked="fk_i === 0 ? true : false" />
            <div class="tab-content border-base-300 bg-base-100 pt-1 px-5">
                <div class="flex flex-col w-10/10 h-10/10 overflow-y-auto">
                    <div v-for="(fundObj, fund_i) in funds[fundKey]" class="chat"
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
                </div>
            </div>    
        </template>
    </div>
</div>

<div class="w-10/10 h-1/10">
    &nbsp;
</div>

<div class="w-10/10 h-1/10 flex flex-col text-3xl justify-center items-center" :class="{'bg-gray-200': total_fund === 0, 'bg-green-200': total_fund > 0, 'bg-red-200': total_fund < 0}">
    結餘：$ {{ new Intl.NumberFormat().format( total_fund ) }}
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
