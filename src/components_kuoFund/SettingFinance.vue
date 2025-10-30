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
        code_name: "",
        type: "IN",
        money: 0,
        date: moment().format("YYYY-MM-DD"),
        memo: "",
    });
    let members = reactive([]);

    // 初始化 component
    function init(){
        //console.log("Setting_Finance.init");
        //console.log("props.app_state", props.app_state);
        //console.log("props.title", props.title);
        //console.log("props.account", props.account);

        fetchMembers();
    }

    // 取得 finance 資料
    function fetchMembers(){
        let fetchMembersPromise = fetchData({
            api: "get_members_kf",
        }, "KUO-FUNDS");
        Promise.all([fetchMembersPromise]).then((values) => {
            console.log("fetchMembersPromise.values=", values[0]);

            // 郭家基金 - 成員
            members.splice(0, members.length);

            let memObjs = values[0];
            memObjs.sort((x, y) => {
                if(x["code_name"] > y["code_name"]) return 1;
                if(x["code_name"] < y["code_name"]) return -1;
                if(x["code_name"] === y["code_name"]) return 0;
            });
            memObjs.forEach((memObj, m_i) => {
                members.push(memObj);
            });
        });
    }    
    // 新增 financeObj 單筆資料
    function newRecord(){
        console.log("newRecord.financeObj=", financeObj);

        let newFinanceKFPromise = fetchData({
            api: "new_finance",
            data: {
                finance: financeObj,
            }
        }, "KUO-FUNDS");
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
                code_name: memObj.code_name,
                type: "IN",
                money: financeObj.money,
                date: financeObj.date,
                memo: financeObj.memo,
            };

            //console.log("newFundObj=", newFundObj);
            let newFinanceKFPromise = fetchData({
                api: "new_finance",
                data: {
                    finance: newFundObj,
                }
            }, "KUO-FUNDS");
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

    <div class="w-10/10 flex flex-row gap-2 justify-center">
        <label class="label">日期:</label>
        <input type="date" class="input w-8/10" placeholder="" v-model="financeObj.date" />
    </div>
    
    <div class="w-10/10 flex flex-col md:flex-row gap-2 mt-2">
        <!-- 單一人員 -->
        <div class="w-10/10 md:w-5/10 flex flex-col bg-zinc-200">
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
                    <select class="select w-10/10" v-model="financeObj.code_name">
                        <option v-for="(memObj, m_i) in members" :value="memObj.code_name">{{ memObj.name }}</option>
                    </select>
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
        <div class="w-10/10 md:w-5/10 flex flex-col bg-stone-200">
            <div class="divider">
                全部人員-存款紀錄                
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
