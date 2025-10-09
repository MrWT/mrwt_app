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

    let financeObj = reactive({
        name: "",
        type: "IN",
        money: 0,
        date: moment().format("YYYY-MM-DD"),
        memo: "",
    });
    let members = reactive([]);

    // 初始化 component
    function init(){
        console.log("Setting_Finance.init");
        console.log("props.app_state", props.app_state);
        console.log("props.title", props.title);
        console.log("props.account", props.account);

        fetchFinance();
    }

    // 取得 finance 資料
    function fetchFinance(){
        let fetchAppSetting = fetchData({
            api: "get_app_setting",
        });
        Promise.all([fetchAppSetting]).then((values) => {
            let appSettingObj = values[0];

            // 郭家基金 - 成員
            members.splice(0, members.length);
            appSettingObj["kf_member"].value1.split(",").forEach((member, m_i) => {
                members.push(member);
            });
        });
    }    
    // 新增 financeObj 單筆資料
    function newRecord(){
        console.log("newRecord");

        let newFinanceKFPromise = fetchData({
            api: "new_kuo_funds",
            data: {
                finance: financeObj,
            }
        });
        Promise.all([newFinanceKFPromise]).then((values) => {
            console.log("newFinanceKFPromise.values=", values);

            let opObj = {
                message: "",
                status: true,
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
    // 新增 financeObj 單月多筆資料
    function newOneMonthManyRecord(){
        console.log("newOneMonthManyRecord.members=", members);
        
        members.forEach((memObj, mem_i) => {
            let newFundObj = {
                name: memObj,
                type: "IN",
                money: financeObj.money,
                date: financeObj.date,
                memo: financeObj.memo,
            };

            //console.log("newFundObj=", newFundObj);
            let newFinanceKFPromise = fetchData({
                api: "new_kuo_funds",
                data: {
                    finance: newFundObj,
                }
            });
            Promise.all([newFinanceKFPromise]).then((values) => {
                console.log("newFinanceKFPromise.values=", values);

                let opObj = {
                    message: "",
                    status: true,
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
        });
        
    }
   
</script>

<template>

    <div class="w-10/10 flex flex-col">
        <label class="label">日期:</label>
        <input type="date" class="input w-10/10" placeholder="" v-model="financeObj.date" />
    </div>
    
    <div class="w-10/10 flex flex-row gap-1">
        <!-- 單一人員 -->
        <div class="w-5/10 flex flex-col bg-zinc-200">
            <div class="divider">
                單一人員
            </div>
            <div class="w-10/10 flex flex-col gap-3 place-items-center">
                <div class="flex flex-row gap-2 w-7/10">
                    <label class="text-green-900 text-lg w-5/10">
                        <input type="radio" v-model="financeObj.type" value="IN" />
                        存款
                    </label>
                    <label class="text-red-900 text-lg w-5/10">
                        <input type="radio" v-model="financeObj.type" value="OUT" />
                        提領
                    </label>
                </div>

                <div class="w-7/10 flex flex-col">
                    <label class="label">人員姓名:</label>
                    <input type="text" class="input w-10/10" placeholder="" v-model="financeObj.name" list="members" />
                    <datalist id="members">
                        <option v-for="(member, m_i) in members" :value="member"></option>
                    </datalist>
                </div>

                <div class="w-7/10 flex flex-col">
                    <label class="label">金額:</label>
                    <input type="number" min="0" class="input w-10/10" placeholder="" v-model="financeObj.money" />
                </div>
                
                <div class="w-7/10 flex flex-col">
                    <label class="label">備註</label>
                    <input type="text" class="input w-10/10" placeholder="" v-model="financeObj.memo" />
                </div>
            </div>
            <div class="w-10/10 flex flex-col md:flex-row-reverse mt-5 justify-center">
                <button class="btn btn-neutral w-10/10 md:w-5/10" @click="newRecord">
                    NEW
                </button>
            </div>

            <div class="divider"></div>
        </div>
        <!-- 全部人員 -->
        <div class="w-5/10 flex flex-col bg-stone-200">
            <div class="divider">
                全部人員
            </div>
            <div class="w-10/10 flex flex-col gap-3 place-items-center">
                <div class="w-7/10 flex flex-col">
                    <label class="label">金額:</label>
                    <input type="number" min="0" class="input w-10/10" placeholder="" v-model="financeObj.money" />
                </div>
                
                <div class="w-7/10 flex flex-col">
                    <label class="label">備註</label>
                    <input type="text" class="input w-10/10" placeholder="" v-model="financeObj.memo" />
                </div>
            </div>
            <div class="w-10/10 flex flex-col md:flex-row-reverse mt-5 justify-center">
                <button class="btn btn-neutral w-10/10 md:w-5/10" @click="newOneMonthManyRecord">
                    NEW
                </button>
            </div>

            <div class="divider"></div>
        </div>
    </div>

    

    

</template>

<style scoped>

</style>
