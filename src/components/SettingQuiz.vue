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
        console.log("Setting_Quiz mounted.");
        init();
    });

    let quiz_setting = reactive({
        max_number: 0,
        count: 0,
    });

    // 初始化 component
    function init(){
        console.log("Setting_Quiz.init");
        console.log("props.app_state", props.app_state);
        console.log("props.title", props.title);
        console.log("props.account", props.account);

        fetchQuiz();
    }
    // 取得 quiz 預設值
    function fetchQuiz(){
        let fetchAppSetting = fetchData({
            api: "get_app_setting",
        });
        Promise.all([fetchAppSetting]).then((values) => {
            let appSettingObj = values[0];

            quiz_setting.max_number = appSettingObj["quiz_max_number"];
            quiz_setting.count = appSettingObj["quiz_count"];
        });
    }    
    // 儲存 quiz 設定資料
    function saveQuiz(){
        console.log("saveQuiz");

        let saveQuizPromise = fetchData({
            api: "save_quiz",
            data: quiz_setting
        });
        Promise.all([saveQuizPromise]).then((values) => {
            console.log("saveQuizPromise.values=", values);

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
    
</script>

<template>

    <div class="divider">
        Quiz 預設值
    </div>
    <div class="flex w-10/10 flex-col gap-5 place-items-center">
        <div class="w-10/10 md:w-5/10">
            <label class="label mr-1">題數: </label>
            <input type="number" min="0" class="input" placeholder="0" v-model="quiz_setting.count" />
        </div>
        <div class="w-10/10 md:w-5/10">
            <label class="label mr-1">最大的數字: </label>
            <input type="number" min="0" class="input" placeholder="0" v-model="quiz_setting.max_number" />
        </div>
    </div>
    <div class="w-10/10 flex flex-col md:flex-row-reverse mt-1 justify-center">
        <button class="btn btn-neutral w-10/10 md:w-5/10 mx-1" @click="saveQuiz">
            save
        </button>
    </div>

</template>

<style scoped>

</style>
