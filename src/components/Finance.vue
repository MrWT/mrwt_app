<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import { fetchData } from "@/composables/fetchData"

    import SettingFinance from '@/components/SettingFinance.vue'

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        user_role: String,
    });

    onMounted(() => {
        console.log("Finance mounted.");
        init();
    });

    let appState = ref("");
    let progressSetting = reactive({
        house: {
            value: null,
            max: null,
            targetText: "",
            progressText: "",
        },
        credit: {
            value: null,
            max: null,
            targetText: "",
            progressText: "",
            remainText: "",
            remainValue: null,
        },
        speed: {
            target: 0,
            speed: 0,
            perMonth: 0,
        },
    });
    let deposit_TWD = ref(0);
    let deposit_USD_insurance = ref(0);
    let deposit_LikeTWD_insurance = ref(0);
    let deposit_USD_fixed = ref(0);
    let deposit_LikeTWD_fixed = ref(0);
    let stock_USD = ref(0);
    let stock_LikeTWD = ref(0);

    let stockTW = reactive({
        totalValue: "",
        totalTWD: "",
        tw0056: "",
        tw0056_TWD: "",
        tw00878: "",
        tw00878_TWD: "",
        tw00919: "",
        tw00919_TWD: "",
    });

    // 初始化 component
    function init(){
        console.log("finance.init");
        console.log("props.title=" + props.title);
        console.log("props.account=" + props.account);
        console.log("props.user_role=" + props.user_role);

        if(props.account){
            // 取得使用者個人 finance 資料
            fetchFinance();
        }
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

            let stockDatas_TWD = [];
            let stockData_USD = null;
            let houseObj = null;
            let creditObj = null;
            let depositObj_TWD = null;
            let depositObj_USD_insurance = null;
            let depositObj_USD_fixed = null;
            let depositObj_Speed = null;
            values[0].forEach((finObj, fin_i) => {
                if(finObj["name"] === "house"){
                    // 購屋資訊
                    houseObj = finObj;
                } else if(finObj["name"] === "credit"){
                    // 信貸資訊
                    creditObj = finObj;
                } else if(finObj["name"].indexOf("stock_") >= 0 && finObj["name"] !== "stock_nano"){
                    // 台股資訊
                    stockDatas_TWD.push( finObj  );
                } else if(finObj["name"] === "stock_nano"){
                    // 奈米投資訊
                    stockData_USD = finObj;
                } else if(finObj["name"] === "deposit" && finObj["currency"] === "TWD"){
                    // 台幣存款資訊
                    depositObj_TWD = finObj;
                } else if(finObj["name"] === "deposit_insurance" && finObj["currency"] === "USD"){
                    // 美金存款資訊 - 保險
                    depositObj_USD_insurance = finObj;
                } else if(finObj["name"] === "deposit_fixed" && finObj["currency"] === "USD"){
                    // 美金存款資訊 - 定存
                    depositObj_USD_fixed = finObj;
                } else if(finObj["name"] === "speed"){
                    // 存款速度資訊
                    depositObj_Speed = finObj;
                }
            });
            buildStockTW(stockDatas_TWD);
            buildStockGlobal(stockData_USD);
            buildDepositTWD(depositObj_TWD);
            buildDepositUSD_insurance(depositObj_USD_insurance);
            buildDepositUSD_fixed(depositObj_USD_fixed);
            buildCreditBlock(creditObj, depositObj_TWD);
            buildHouseBlock(houseObj, depositObj_USD_insurance, depositObj_USD_fixed,  stockData_USD);
            buildSpeedBlock(depositObj_Speed);
        });
    }
    // 建立"購屋進度"區塊
    function buildHouseBlock(houseObj, depositObj_USD_insurance, depositObj_USD_fixed, stockData_USD){
        let targetValue = houseObj["value1"];
        let currentValue = depositObj_USD_insurance["value1"] * depositObj_USD_insurance["value2"] 
                        + depositObj_USD_fixed["value1"] * depositObj_USD_fixed["value2"] 
                        + stockData_USD["value1"] * stockData_USD["value2"];

        progressSetting.house.max = 100;
        progressSetting.house.value = Math.floor( currentValue * 100 / (targetValue * 0.3) );

        progressSetting.house.targetText = "購屋目標: " + (new Intl.NumberFormat().format(targetValue));
        progressSetting.house.progressText = "";
        progressSetting.house.progressText += "進度: ";
        progressSetting.house.progressText += (new Intl.NumberFormat().format(currentValue)) + " / ";
        progressSetting.house.progressText += (new Intl.NumberFormat().format(targetValue * 0.3)) + "( 粗估三成 )";
        progressSetting.house.progressText += " = " + progressSetting.house.value + "%";
    }
    // 建立"還款進度"區塊
    function buildCreditBlock(creditObj, depositObj_TWD){
        let targetValue = creditObj["value1"];
        let currentValue = creditObj["value2"];
        let depositValue = depositObj_TWD["value1"];

        progressSetting.credit.max = 100;
        progressSetting.credit.value = Math.floor( (targetValue - currentValue) * 100 / targetValue );

        progressSetting.credit.targetText = "還款目標: " + (new Intl.NumberFormat().format(targetValue));
        progressSetting.credit.progressText = "";
        progressSetting.credit.progressText += "進度: ";
        progressSetting.credit.progressText += (new Intl.NumberFormat().format(targetValue - currentValue)) + " / ";
        progressSetting.credit.progressText += (new Intl.NumberFormat().format(targetValue));
        progressSetting.credit.progressText += "=" + progressSetting.credit.value + "%";

        progressSetting.credit.remainValue = Math.floor((targetValue - (currentValue - depositValue))*100 / targetValue);
        progressSetting.credit.remainText = "";
        progressSetting.credit.remainText += "剩餘: " + (new Intl.NumberFormat().format(currentValue))
                                                + "-" + (new Intl.NumberFormat().format(depositValue))
                                                + "=" + (new Intl.NumberFormat().format(currentValue - depositValue)) + "";
        progressSetting.credit.remainText += "( " + progressSetting.credit.remainValue + "% )";
    }
    // 建立"存款速度"區塊
    function buildSpeedBlock(depositData){
        //console.log("buildSpeedBlock.depositData=", depositData);

        let targetValue = 100 * 10000; // 目標: 存到 100 萬台幣的速度
        let twdValuePer3Month = parseInt( depositData["value1"] ); // 每期可以存到的台幣( 每期是 3 個月 )
        let stockValuePer3Month = parseInt( depositData["value2"] ); // 每期可以存到的股息( 每期是 3 個月 )
        let valuePerMonth = Math.floor( ( twdValuePer3Month + stockValuePer3Month ) / 3 ); // 每個月共可存到的台幣
        let valuePer3Month = ( twdValuePer3Month + stockValuePer3Month ); // 每期共可存到的台幣

        let speed = Math.ceil( targetValue / valuePer3Month );

        console.log("buildSpeedBlock.targetValue=" + targetValue);
        console.log("buildSpeedBlock.valuePer3Month=" + valuePer3Month);
        console.log("buildSpeedBlock.valuePerMonth=" + valuePerMonth);
        console.log("buildSpeedBlock.speed=" + speed);

        progressSetting["speed"]["target"] = Math.floor( targetValue / 10000 );
        progressSetting["speed"]["speed"] = speed;
        progressSetting["speed"]["perMonth"] = (new Intl.NumberFormat().format(valuePerMonth)); 
        progressSetting["speed"]["per3Month"] = (new Intl.NumberFormat().format(valuePer3Month)); 
        progressSetting["speed"]["twdValuePer3Month"] = (new Intl.NumberFormat().format(twdValuePer3Month));
        progressSetting["speed"]["stockValuePer3Month"] = (new Intl.NumberFormat().format(stockValuePer3Month));
    }
    // 建立"台灣股票"區塊
    function buildStockTW(stockDatas){
        console.log("buildStockTW.stockDatas=", stockDatas);

        let stockTotalValue = 0;
        let stockTotalTWD = 0;
        let stockLabels = [];
        let stockSeries = [];
        stockDatas.forEach((dataObj, d_i) => {
            let name = dataObj["name"];
            let stockLabel = name.split("_")[1];

            stockLabels.push( stockLabel );
            stockSeries.push( dataObj["value1"] );

            stockTotalValue += dataObj["value1"];
            stockTotalTWD += (dataObj["value1"] * dataObj["value2"]);
            switch(dataObj["name"]){
                case "stock_0056":
                    stockTW.tw0056 = new Intl.NumberFormat().format( dataObj["value1"] );
                    stockTW.tw0056_TWD = new Intl.NumberFormat().format( dataObj["value1"] * dataObj["value2"] );
                    break;
                case "stock_00878":
                    stockTW.tw00878 = new Intl.NumberFormat().format( dataObj["value1"] );
                    stockTW.tw00878_TWD = new Intl.NumberFormat().format( dataObj["value1"] * dataObj["value2"] );
                    break;
                case "stock_00919":
                    stockTW.tw00919 = new Intl.NumberFormat().format( dataObj["value1"] );
                    stockTW.tw00919_TWD = new Intl.NumberFormat().format( dataObj["value1"] * dataObj["value2"] );
                    break;
            }
        });
        stockTW.totalValue = new Intl.NumberFormat().format(stockTotalValue);
        stockTW.totalTWD = new Intl.NumberFormat().format(stockTotalTWD);
    }
    // 建立"全球股票"區塊
    function buildStockGlobal(stockData){
        console.log("buildStockGlobal.stockData=", stockData);

        stock_USD.value = new Intl.NumberFormat('en-US').format(stockData["value1"]);
        stock_LikeTWD.value = new Intl.NumberFormat('en-US').format(stockData["value1"] * stockData["value2"]);
    }
    // 建立"台幣存款"區塊
    function buildDepositTWD(depositData){
        console.log("buildDepositTWD.depositData=", depositData);

        deposit_TWD.value = new Intl.NumberFormat('en-US').format(depositData["value1"]);
    }
    // 建立"美金存款"區塊-保險
    function buildDepositUSD_insurance(depositData){
        console.log("buildDepositUSD_insurance.depositData=", depositData);
        deposit_USD_insurance.value = new Intl.NumberFormat('en-US').format(depositData["value1"]);
        deposit_LikeTWD_insurance.value = new Intl.NumberFormat('en-US').format(depositData["value1"] * depositData["value2"]);
    }
    // 建立"美金存款"區塊-定存
    function buildDepositUSD_fixed(depositData){
        console.log("buildDepositUSD_fixed.depositData=", depositData);
        deposit_USD_fixed.value = new Intl.NumberFormat('en-US').format(depositData["value1"]);
        deposit_LikeTWD_fixed.value = new Intl.NumberFormat('en-US').format(depositData["value1"] * depositData["value2"]);
    }
    // 開啟 setting modal
    function openSettingModal(){
        document.getElementById("settingModal").showModal();
    }
    // 關閉 setting modal
    function closeSettingModal(){
        document.getElementById("settingModal").close();
    }
    // 給 Setting Modal 使用的 function
    function modalStatus(opMode, message){
        if(opMode === "SAVE_SUCCESS"){
            emit('popupMessage', true, message); // Emitting the event with data
            // 更新資料
            fetchFinance();
        }else if(opMode === "SAVE_FAIL"){
            emit('popupMessage', false, message); // Emitting the event with data
        }

        closeSettingModal();
    }


</script>

<template>

<div class="flex flex-col w-1/1 h-1/1 gap-2">
    <div v-if="props.user_role === 'admin'" class="flex flex-row justify-end w-1/1 bg-transparent rounded-xl gap-2">
        <a class="text-gray-500 hover:text-gray-900 cursor-pointer" @click="openSettingModal">
            <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 13v-2a1 1 0 0 0-1-1h-.757l-.707-1.707.535-.536a1 1 0 0 0 0-1.414l-1.414-1.414a1 1 0 0 0-1.414 0l-.536.535L14 4.757V4a1 1 0 0 0-1-1h-2a1 1 0 0 0-1 1v.757l-1.707.707-.536-.535a1 1 0 0 0-1.414 0L4.929 6.343a1 1 0 0 0 0 1.414l.536.536L4.757 10H4a1 1 0 0 0-1 1v2a1 1 0 0 0 1 1h.757l.707 1.707-.535.536a1 1 0 0 0 0 1.414l1.414 1.414a1 1 0 0 0 1.414 0l.536-.535 1.707.707V20a1 1 0 0 0 1 1h2a1 1 0 0 0 1-1v-.757l1.707-.708.536.536a1 1 0 0 0 1.414 0l1.414-1.414a1 1 0 0 0 0-1.414l-.535-.536.707-1.707H20a1 1 0 0 0 1-1Z"/>
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15a3 3 0 1 0 0-6 3 3 0 0 0 0 6Z"/>
            </svg>
        </a>
    </div>

    <div class="flex flex-col w-1/1 gap-2">
        <div class="flex flex-row w-1/1 h-1/1">
            <div class="card rounded-box grid h-1/1 w-1/1 p-5 place-items-center"
                 :class="{'bg-red-300': 12 <= progressSetting.speed.speed,
                          'bg-orange-300': 7 <= progressSetting.speed.speed && progressSetting.speed.speed < 12,
                          'bg-blue-300': 3 < progressSetting.speed.speed && progressSetting.speed.speed < 7,
                          'bg-green-300': progressSetting.speed.speed <= 3}">
                <div class="w-1/1 text-xl text-center">
                    約需 {{ progressSetting["speed"]["speed"] }} 期<br />
                    可以存到 {{ progressSetting["speed"]["target"] }} 萬
                </div>
            </div>
        </div>
        <div class="flex flex-col md:flex-row w-10/10 h-10/10 gap-2">        
            <div class="card rounded-box grid h-1/1 w-1/1 md:w-1/2 p-5 place-items-center"
                 :class="{'bg-red-300': 12 <= progressSetting.speed.speed,
                          'bg-orange-300': 7 <= progressSetting.speed.speed && progressSetting.speed.speed < 12,
                          'bg-blue-300': 3 < progressSetting.speed.speed && progressSetting.speed.speed < 7,
                          'bg-green-300': progressSetting.speed.speed <= 3}">
                <div class="w-10/10 text-xl text-center">
                    每期(薪資+股利)約可存<br />
                    TWD ${{ progressSetting["speed"]["per3Month"] }}
                </div>
            </div>
            <div class="h-1/1 flex content-center hidden md:block">
                <div>=</div>
            </div>
            <div class="flex flex-row w-1/1 md:w-1/2 gap-2">
                <div class="card rounded-box grid h-1/1 w-1/2 p-5 place-items-center bg-base-300 ">
                    <div class="w-10/10 text-md text-center">
                        每期(薪資)約可存<br />
                        TWD ${{ progressSetting["speed"]["twdValuePer3Month"] }}
                    </div>
                </div>
                <div class="h-1/1 flex items-center">
                    <div>+</div>
                </div>
                <div class="card rounded-box grid h-1/1 w-1/2 p-5 place-items-center bg-base-300 ">
                    <div class="w-10/10 text-md text-center">
                        每期(股利)約可存<br />
                        TWD ${{ progressSetting["speed"]["stockValuePer3Month"] }}
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="divider">股票</div>
    <div class="flex flex-col md:flex-row w-1/1 h-1/1 gap-2">
        <div class="bg-base-300 rounded-box flex flex-col p-5 h-10/10 w-1/1 md:w-1/2 items-center">
            <span class="h-1/1 inline-block align-middle">
                <span class="text-2xl">總價值(TWD): {{ stockTW.totalTWD }}</span>
            </span>
        </div>
        <div class="h-1/1 flex content-center hidden md:block">
            <div>=</div>
        </div>
        <div class="flex flex-row gap-1 w-1/1 md:w-1/2">
            <div class="card bg-gray-300 rounded-box grid p-5 h-10/10 w-1/2 place-items-start">
                <span class="text-2xl">0056 </span> 
                <span class="text-lg">股數: {{ stockTW.tw0056 }}</span>
                <span class="text-lg">TWD: {{ stockTW.tw0056_TWD }}</span>
            </div>
            <div class="h-1/1 flex items-center">
                <div>+</div>
            </div>
            <div class="card bg-gray-300 rounded-box grid p-5 h-10/10 w-1/2 place-items-start">
                <span class="text-2xl">00878 </span>
                <span class="text-lg">股數: {{ stockTW.tw00878 }}</span>
                <span class="text-lg">TWD: {{ stockTW.tw00878_TWD }}</span>
            </div>
        </div>
    </div>

    <div class="divider">TWD 計價</div>
    <div class="card bg-base-300 rounded-box grid h-3/10 w-10/10 p-5 place-items-center">
        <div class="w-1/1 text-2xl text-center">存款: {{ deposit_TWD }}</div>
    </div>

    <div class="card bg-base-300 rounded-box grid h-10/10 w-10/10 p-5 place-items-center mt-1">
        <div class="flex flex-col w-10/10">
            <span class="text-2xl">{{ progressSetting.credit.targetText }}</span>
            <span class="text-lg">{{ progressSetting.credit.progressText }}</span>
        </div>
        <progress class="w-10/10 progress"
                :class="{ 'progress-error': progressSetting.credit.value && progressSetting.credit.value < 50,
                            'progress-warning': progressSetting.credit.value && 50 <= progressSetting.credit.value && progressSetting.credit.value < 80,
                            'progress-info': progressSetting.credit.value && 80 <= progressSetting.credit.value && progressSetting.credit.value < 90,
                            'progress-success': progressSetting.credit.value && 90 <= progressSetting.credit.value }"
                :value="progressSetting.credit.value" :max="progressSetting.credit.max">
        </progress>
        <div class="flex flex-col w-10/10">
            <span>{{ progressSetting.credit.remainText }}</span>
        </div>
        <progress class="w-10/10 progress"
                :class="{ 'progress-error': progressSetting.credit.remainValue && progressSetting.credit.remainValue < 50,
                            'progress-warning': progressSetting.credit.remainValue && 50 <= progressSetting.credit.remainValue && progressSetting.credit.remainValue < 80,
                            'progress-info': progressSetting.credit.remainValue && 80 <= progressSetting.credit.remainValue && progressSetting.credit.remainValue < 90,
                            'progress-success': progressSetting.credit.remainValue && 90 <= progressSetting.credit.remainValue }"
                :value="progressSetting.credit.remainValue" :max="progressSetting.credit.max">
        </progress>
    </div>

    <div class="card bg-base-300 rounded-box grid h-10/10 w-10/10 p-5 place-items-center mt-1">
        <div class="flex flex-col w-10/10">
            <span class="text-2xl">{{ progressSetting.house.targetText }}</span>
            <span class="text-lg">{{ progressSetting.house.progressText }}</span>
        </div>
        <progress class="w-10/10 progress"
                :class="{ 'progress-error': progressSetting.house.value && progressSetting.house.value < 50,
                            'progress-warning': progressSetting.house.value && 50 <= progressSetting.house.value && progressSetting.house.value < 80,
                            'progress-info': progressSetting.house.value && 80 <= progressSetting.house.value && progressSetting.house.value < 90,
                            'progress-success': progressSetting.house.value && 90 <= progressSetting.house.value }"
                :value="progressSetting.house.value" :max="progressSetting.house.max">
        </progress>
    </div>

    <div class="divider">USD 計價</div>
    
    <div class="flex flex-col md:flex-row w-10/10 h-10/10 gap-2">
        <div class="card bg-base-300 rounded-box grid h-10/10 w-10/10 md:w-5/10 p-5 place-items-center">
            <div class="w-10/10 text-2xl">保險 USD: {{ deposit_USD_insurance }}</div>
            <div class="w-10/10 text-lg">TWD( 1:30 ): {{ deposit_LikeTWD_insurance }}</div>
        </div>

        <div class="card bg-base-300 rounded-box grid h-10/10 w-10/10 md:w-5/10 p-5 place-items-center">
            <div class="w-10/10 text-2xl">定存 USD: {{ deposit_USD_fixed }}</div>
            <div class="w-10/10 text-lg">TWD( 1:30 ): {{ deposit_LikeTWD_fixed }}</div>
        </div>

        <div class="card bg-base-300 rounded-box grid h-10/10 w-10/10 md:w-5/10 p-5 place-items-center">
            <div class="w-10/10 text-2xl">奈米投 USD: {{ stock_USD }}</div>
            <div class="w-10/10 text-lg">TWD( 1:30 ): {{ stock_LikeTWD }}</div>
        </div>
    </div>

    <div class="divider"></div>
</div>

<!-- setting modal -->
<dialog id="settingModal" class="modal">
    <div class="modal-box h-9/10 w-1/1 flex flex-col bg-neutral-100">
        <SettingFinance :title="props.title" :account="props.account" @modal-status="modalStatus" />
    </div>
</dialog>

</template>

<style scoped>

</style>
