<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("Finance_KF mounted.");
        init();
    });

    let appState = ref("");
    let total_fund = ref(0);
    let funds_202510 = reactive([]);
    let funds_202511 = reactive([]);

    // 初始化 component
    function init(){
        console.log("Finance_KF.init");
        console.log("Finance_KF.props.title", props.title);
        console.log("Finance_KF.props.account", props.account);

        fetchFunds();
    }    

    function fetchFunds(){
        console.log("fetchFunds");

         let fetchFundsPromise = fetchData({
            api: "get_kuo_funds",
        });
        Promise.all([fetchFundsPromise]).then((values) => {
            console.log("fetchFundsPromise.values=", values);

            funds_202510.splice(0, funds_202510.length);
            funds_202511.splice(0, funds_202511.length);
            values[0].forEach((fundObj, fund_i) => {
                if(moment(fundObj["date"]).format("YYYYMM") === "202510"){
                    funds_202510.push(fundObj);

                    if(fundObj["type"] === "IN"){
                        total_fund.value += fundObj["money"];
                    }else{
                        total_fund.value -= fundObj["money"];
                    }
                }else if(moment(fundObj["date"]).format("YYYYMM") === "202511"){
                    funds_202511.push(fundObj);

                    if(fundObj["type"] === "IN"){
                        total_fund.value += fundObj["money"];
                    }else{
                        total_fund.value -= fundObj["money"];
                    }
                }

            });

            console.log("funds_202510=", funds_202510);
            console.log("funds_202511=", funds_202511);
            console.log("total_fund=", total_fund.value);
        });

    }
    
</script>

<template>

<div class="tabs tabs-border">
    <!-- 202510 -->
    <input type="radio" name="funds_tabs" class="tab" aria-label="2025-10" checked />
    <div class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <div class="flex flex-col w-10/10 h-8/10 overflow-y-auto">
            <div v-for="(fundObj, fund_i) in funds_202510" class="chat"
                :class="{ 'chat-start': fundObj.type === 'IN', 'chat-end': fundObj.type === 'OUT' }">
                <div class="chat-image avatar">
                    <div class="avatar avatar-placeholder">
                        <div class="w-12 rounded-full"
                            :class="{'bg-green-900': fundObj.type === 'IN', 'bg-red-900': fundObj.type === 'OUT'}">
                            <span v-if="fundObj.type === 'IN'" class="text-lg text-white">進帳</span>
                            <span v-if="fundObj.type === 'OUT'" class="text-lg text-white">出帳</span>
                        </div>
                    </div>
                </div>
                <div class="chat-header">
                    <span class="text-base">{{ fundObj.name }}</span>
                    <time class="text-base opacity-50">{{ fundObj.date }}</time>
                </div>
                <div class="chat-bubble">{{ fundObj.money }}</div>
            </div>
        </div>
    </div>

    <!-- 202511 -->
    <input type="radio" name="funds_tabs" class="tab" aria-label="2025-11" />
    <div class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <div class="flex flex-col w-10/10 h-8/10 overflow-y-auto">
            <div v-for="(fundObj, fund_i) in funds_202511" class="chat"
                :class="{ 'chat-start': fundObj.type === 'IN', 'chat-end': fundObj.type === 'OUT' }">
                <div class="chat-image avatar">
                    <div class="avatar avatar-placeholder">
                        <div class="w-12 rounded-full"
                            :class="{'bg-green-900': fundObj.type === 'IN', 'bg-red-900': fundObj.type === 'OUT'}">
                            <span v-if="fundObj.type === 'IN'" class="text-lg text-white">進帳</span>
                            <span v-if="fundObj.type === 'OUT'" class="text-lg text-white">出帳</span>
                        </div>
                    </div>
                </div>
                <div class="chat-header">
                    <span class="text-base">{{ fundObj.name }}</span>
                    <time class="text-base opacity-50">{{ fundObj.date }}</time>
                </div>
                <div class="chat-bubble">{{ fundObj.money }}</div>
            </div>
        </div>
    </div>

</div>

<div class="w-10/10 h-2/10 flex flex-col text-3xl justify-center items-center" :class="{'bg-gray-200': total_fund === 0, 'bg-green-200': total_fund > 0, 'bg-red-200': total_fund < 0}">
    帳戶餘額：{{ total_fund }}
</div>

</template>

<style scoped>

</style>
