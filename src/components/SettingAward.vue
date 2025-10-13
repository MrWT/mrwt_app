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
        console.log("Setting_Award mounted.");
        init();
    });
   
    let awards = reactive([]);

    // 初始化 component
    function init(){
        //console.log("Setting_Award.init");
        //console.log("props.app_state", props.app_state);
        //console.log("props.title", props.title);
        //console.log("props.account", props.account);

        fetchAward();
    }
    // 取得 award 資料
    function fetchAward(){
        let fetchAwardPromise = fetchData({
            api: "get_awards",
            data: {
                check_receive_count: "false",
            },
        });
        Promise.all([fetchAwardPromise]).then((values) => {
            console.log("fetchAwardPromise.values=", values);

            // 清空 awards
            awards.splice(0, awards.length);
            values[0].forEach((awardObj, award_i) => {
                awards.push({
                    op: "EDIT",
                    name: awardObj["name"],
                    display_text: awardObj["display_text"],
                    count: awardObj["count"],
                    pickup_percent: awardObj["pickup_percent"],
                    receive_count: awardObj["receive_count"],
                });
            });
            console.log("awards", awards);
        });
    }
    // 新增 award 設定資料
    function newAward(){
        //console.log("newAward");
        const guid = crypto.randomUUID();

        awards.push({
            op: "NEW",
            name: guid,
            display_text: "",
            count: 0,
            pickup_percent: 0,
            receive_count: 0,
        });
    }
    // 儲存 award 設定資料
    function saveAward(){
        console.log("saveAward.awards=", awards);

        let saveAwardsPromise = fetchData({
            api: "edit_awards",
            data: awards
        });
        Promise.all([saveAwardsPromise]).then((values) => {
            console.log("saveAwardsPromise.values=", values);

            let opObj = {
                message: "",
                status: true,
            };
            opObj.status = values[0]["result"];
            if(values[0]["result"] === true){
                opObj.message = "儲存成功";
                // 更新資料
                fetchAward();
            }else{
                opObj.message = values[0]["message"];
            }

            // 將 message 傳給 Setting.vue 
            emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data
        });
    }
    
</script>

<template>

    <div class="divider">
        Award 資料
    </div>
    <div class="flex w-10/10 flex-col gap-2">
        <fieldset v-for="(awardObj, award_i) in awards" class="fieldset bg-gray-300 border-gray-500 rounded-box border p-2 w-10/10">
            <div class="flex flex-col join-item w-10/10">
                <label class="label">名稱</label>
                <input type="text" class="input w-10/10" placeholder="名稱" v-model="awardObj.display_text" />
            </div>
            <div class="join">
                <div class="flex flex-col join-item w-5/10">
                    <label class="label">總量</label>
                    <input type="number" class="input w-10/10" placeholder="總量" v-model="awardObj.count" />
                </div>
                <div class="flex flex-col join-item w-5/10">
                    <label class="label">已領量</label>
                    <input type="number" class="input w-10/10" placeholder="已領量" v-model="awardObj.receive_count" />
                </div>
            </div>
            <div class="flex flex-col w-10/10">
                <label class="label">(相對)出現機率: {{ awardObj.pickup_percent }}%</label>
                <input type="range" min="0" max="100" v-model="awardObj.pickup_percent" class="range range-info w-10/10" />
            </div>
        </fieldset>
    </div>
    <div class="w-10/10 flex flex-col md:flex-row-reverse mt-2 gap-2 justify-center">
        <button class="btn btn-neutral w-10/10 md:w-5/10" @click="newAward">
            new
        </button>
        <button class="btn btn-neutral w-10/10 md:w-5/10" @click="saveAward">
            save
        </button>
    </div>

</template>

<style scoped>

</style>
