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
    let funds = reactive({});
    let fundKeys = reactive([]);

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

            // 清空 funds
            Object.keys(funds).forEach(key => delete funds[key]);
            values[0].forEach((fundObj, fund_i) => {
                let fund_date = moment(fundObj["date"]).format("YYYYMM");
                if(Object.keys(funds).indexOf(fund_date) < 0){
                    funds[fund_date] = [];
                }

                funds[fund_date].push( fundObj );
                if(fundObj["type"] === "IN"){
                    total_fund.value += fundObj["money"];
                }else{
                    total_fund.value -= fundObj["money"];
                }
            });

            Object.keys(funds).forEach((fundKey, fk_i) => {
                funds[fundKey].sort((x, y) => {
                    if(moment(x["date"]).format("YYYYMMDD") > moment(y["date"]).format("YYYYMMDD")){
                        return 1;
                    }else{
                        return -1;
                    }
                });
            });

            fundKeys = Object.keys(funds);
            fundKeys.sort((x, y) => {
                return parseInt(y) - parseInt(x);
            });

            console.log("funds=", funds);
            console.log("total_fund=", total_fund.value);
        });

    }
    
</script>

<template>

<div class="w-10/10 md:w-6/10 h-9/10 md:h-8/10 justify-self-center">
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
                                    <span v-if="fundObj.type === 'IN'" class="text-lg text-white">存款</span>
                                    <span v-if="fundObj.type === 'OUT'" class="text-lg text-white">提領</span>
                                </div>
                            </div>
                        </div>
                        <div class="chat-header">
                            <span class="text-base">{{ fundObj.name }}</span>
                            <time class="text-base opacity-50">{{ fundObj.date }}</time>
                        </div>
                        <div class="chat-bubble">
                            {{ new Intl.NumberFormat().format( fundObj.money ) }}
                            <span v-if="fundObj.memo !== ''">( {{ fundObj.memo }} )</span>
                        </div>
                    </div>
                </div>
            </div>    
        </template>
    </div>
</div>

<div class="absolute bottom-2 left-0 w-10/10 h-1/10 flex flex-col text-3xl justify-center items-center" :class="{'bg-gray-200': total_fund === 0, 'bg-green-200': total_fund > 0, 'bg-red-200': total_fund < 0}">
    結餘：{{ new Intl.NumberFormat().format( total_fund ) }}
</div>

</template>

<style scoped>

</style>
