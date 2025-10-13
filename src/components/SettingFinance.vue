<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        app_state: String,
        title: String,
        account: String,
    })

    onMounted(() => {
        console.log("Setting_Finance mounted.");
        init();
    });

    let finance = reactive({
        house: {
            target: 0,
        },
        credit: {
            target: 0,
            remain: 0,
        },
        deposit_Speed:{
            value1: 0,
            value2: 0,
        },
        deposit_TWD:{
            value: 0,
        },
        deposit_USD:{
            value: 0,
            currency: 0,
        },
        stock_nano:{
            value: 0,
            currency: 0,
        },
        stock_tw0056: {
            num: 0,
            price: 0,
        },
        stock_tw00878: {
            num: 0,
            price: 0,
        },
        stock_tw00919: {
            num: 0,
            price: 0,
        },
    });

    // 初始化 component
    function init(){
        console.log("Setting_Finance.init");
        console.log("props.app_state", props.app_state);
        console.log("props.title", props.title);
        console.log("props.account", props.account);

        fetchFinance();
    }
    // 取得使用者個人 finance 資料
    function fetchFinance(){
        let fetchFinancePromise = fetchData({
            api: "get_finance",
            data: {
                account: props.account,
            }
        });
        Promise.all([fetchFinancePromise]).then((values) => {
            console.log("fetchFinancePromise.values=", values);

            values[0].forEach((finObj, fin_i) => {
                if(finObj["name"] === "house"){
                    // 購屋資訊
                    finance.house.target = finObj["value1"];
                } else if(finObj["name"] === "credit"){
                    // 信貸資訊
                    finance.credit.target = finObj["value1"];
                    finance.credit.remain = finObj["value2"];
                } else if(finObj["name"] === "stock_0056"){
                    // 台股資訊
                    finance.stock_tw0056.num = finObj["value1"];
                    finance.stock_tw0056.price = finObj["value2"];
                } else if(finObj["name"] === "stock_00878"){
                    // 台股資訊
                    finance.stock_tw00878.num = finObj["value1"];
                    finance.stock_tw00878.price = finObj["value2"];
                } else if(finObj["name"] === "stock_00919"){
                    // 台股資訊
                    finance.stock_tw00919.num = finObj["value1"];
                    finance.stock_tw00919.price = finObj["value2"];
                } else if(finObj["name"] === "stock_nano"){
                    // 奈米投資訊
                    finance.stock_nano.value = finObj["value1"];
                    finance.stock_nano.currency = finObj["value2"];
                } else if(finObj["name"] === "speed"){
                    // 台幣存款速度
                    finance.deposit_Speed.value1 = finObj["value1"];
                    finance.deposit_Speed.value2 = finObj["value2"];
                } else if(finObj["name"] === "deposit" && finObj["currency"] === "TWD"){
                    // 台幣存款資訊
                    finance.deposit_TWD.value = finObj["value1"];
                } else if(finObj["name"] === "deposit" && finObj["currency"] === "USD"){
                    // 美金存款資訊
                    finance.deposit_USD.value = finObj["value1"];
                    finance.deposit_USD.currency = finObj["value2"];
                }
            });
        });
    }
    // 儲存 finance 設定資料
    function saveFinance(){
        console.log("saveFinance.finance=", finance);

        let saveFinancePromise = fetchData({
            api: "save_finance",
            data: {
                finance: finance,
                account: props.account,
            }
        });
        Promise.all([saveFinancePromise]).then((values) => {
            console.log("saveFinancePromise.values=", values);

            let opObj = {
                status: true,
                message: "",
            };
            opObj.status = values[0]["result"];
            if(values[0]["result"] === true){
                opObj.message = "儲存成功";
            }else{
                opObj.message = values[0]["message"];
            }
            // 將 message 傳給 Setting.vue 
            emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data
        });
    }   
</script>

<template>

    <div class="w-10/10 flex flex-col">
        <div class="divider divider-warning">
            買房
        </div>
        <div class="w-10/10 md:w-5/10">
            <label class="label">目標房價</label>
            <input type="number" min="0" class="input" placeholder="0" v-model="finance.house.target" />
        </div>

        <div class="divider divider-error">
            信貸
        </div>
        <div class="w-10/10 md:w-8/10 flex flex-row">
            <div class="w-5/10">
                <label class="label">目標</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.credit.target" />
            </div>
            <div class="w-5/10">
                <label class="label">剩餘</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.credit.remain" />
            </div>
        </div>

        <div class="divider divider-success">存款</div>
        <div class="divider divider-success">
            存款速度( 以 3 個月為一期 )
        </div>
        <div class="w-10/10 md:w-8/10 flex flex-row">
            <div class="w-5/10">
                <label class="label">台幣/期</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.deposit_Speed.value1" />
            </div>
            <div class="w-5/10">
                <label class="label">股息/期</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.deposit_Speed.value2" />
            </div>
        </div>

        <div class="divider divider-success">
            存款( TWD )
        </div>
        <div class="w-10/10 md:w-5/10 flex flex-row">
            <input type="number" min="0" class="input" placeholder="0" v-model="finance.deposit_TWD.value" />
        </div>

        <div class="divider divider-success">
            存款( USD )
        </div>
        <div class="w-10/10 md:w-8/10 flex flex-row">
            <div class="w-5/10">
                <label class="label">價值</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.deposit_USD.value" />
            </div>
            <div class="w-5/10">
                <label class="label">匯率</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.deposit_USD.currency" />
            </div>
        </div>

        <div class="divider divider-neutral">
            存股
        </div>
        <div class="divider divider-neutral">
            0056.TW
        </div>
        <div class="w-10/10 md:w-8/10 flex flex-row">
            <div class="w-5/10">
                <label class="label">股數</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_tw0056.num" />
            </div>
            <div class="w-5/10">
                <label class="label">單價</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_tw0056.price" />
            </div>
        </div>

        <div class="divider divider-neutral">
            00878.TW
        </div>
        <div class="w-10/10 md:w-8/10 flex flex-row">
            <div class="w-5/10">
                <label class="label">股數</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_tw00878.num" />
            </div>
            <div class="w-5/10">
                <label class="label">單價</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_tw00878.price" />
            </div>
        </div>

        <div class="divider divider-neutral">
            00919.TW
        </div>
        <div class="w-10/10 md:w-8/10 flex flex-row">
            <div class="w-5/10">
                <label class="label">股數</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_tw00919.num" />
            </div>
            <div class="w-5/10">
                <label class="label">單價</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_tw00919.price" />
            </div>
        </div>

        <div class="divider">
            奈米投
        </div>
        <div class="w-10/10 md:w-8/10 flex flex-row">
            <div class="w-5/10">
                <label class="label">USD</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_nano.value" />
            </div>
            <div class="w-5/10">
                <label class="label">匯率</label>
                <input type="number" min="0" class="input" placeholder="0" v-model="finance.stock_nano.currency" />
            </div>
        </div>
    </div>
    <div class="w-10/10 flex flex-col md:flex-row-reverse mt-5 justify-center">
        <button class="btn btn-neutral w-10/10 md:w-5/10" @click="saveFinance">
            save
        </button>
    </div>

</template>

<style scoped>

</style>
