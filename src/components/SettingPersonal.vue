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
        console.log("Setting_Personal mounted.");
        init();
    });

    let userObj = reactive({
        account: "",
        name: "",
        mail: "",
        app_title: "",
        language: "",
        role: "",
    });
    // language radio group 控制項
    let languageRadioParam = reactive({
        radioGroupName: 'rgLang', // 一致的 name 属性
        options: [
            { label: '英文', value: 'EN' },
            { label: '繁體中文', value: 'ZH_TW' }
        ],
        selectedOption: 'EN' // 預設選項
    });

    // 初始化 component
    function init(){
        //console.log("Setting_Personal.init");
        //console.log("props.app_state", props.app_state);
        //console.log("props.title", props.title);
        //console.log("props.account", props.account);

        fetchUser();
    }
    
    // 取得使用者個人資料
    function fetchUser(){
        let fetchUserPromise = fetchData({
            api: "get_user",
            data: {
                account: props.account,
            }
        });
        Promise.all([fetchUserPromise]).then((values) => {
            console.log("fetchUserPromise.values=", values);

            userObj["account"] = props.account;
            if(values.length > 0){
                userObj["name"] = values[0].name;
                userObj["mail"] = values[0].mail;
                userObj["app_title"] = values[0].app_title;
                userObj["language"] = values[0].language;
                userObj["role"] = values[0].role;

                languageRadioParam["selectedOption"] = values[0].language;
            }
        });
    }
    // 儲存個人資料
    function saveUser(){
        console.log("saveUser.userObj=", userObj);
        console.log("saveUser.languageRadioParam=", languageRadioParam);

        let postData = [{
            account: userObj["account"],
            name: userObj["name"],
            mail: userObj["mail"],
            app_title: userObj["app_title"],
            language: languageRadioParam["selectedOption"],
            role: userObj["role"],
        }];

        let saveUserPromise = fetchData({
            api: "save_users",
            data: postData
        });
        Promise.all([saveUserPromise]).then((values) => {
            console.log("saveUserPromise.values=", values);
            
            let opObj = {
                status: true,
                message: "",
            };

            opObj.status = values[0]["result"];
            if(values[0]["result"] === true){
                opObj.message = "儲存成功";
                // 更新資料
                fetchUser();
            }else{
                opObj.message = values[0]["message"];
            }
            // 將 message 傳給 Setting.vue 
            emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data
        });
    }
</script>

<template>

    <div class="flex w-10/10 flex-col place-items-center gap-5">
        <div class="w-10/10 md:w-5/10">
            <label class="label mr-1">Name: </label><br />
            <input type="text" class="input" v-model="userObj.name" />
        </div>
        <div class="w-10/10 md:w-5/10">
            <label class="label mr-1">Mail: </label><br />
            <input type="text" class="input" v-model="userObj.mail" />
        </div>
        <div class="w-10/10 md:w-5/10">
            <label class="label mr-1">Language: </label><br />
            <label v-for="option in languageRadioParam.options" :key="option.value" class="mr-2">
                <input type="radio" class="radio" :value="option.value" :name="languageRadioParam.radioGroupName" v-model="languageRadioParam.selectedOption">
                {{ option.label }}
            </label>
        </div>
        <div class="w-10/10 md:w-5/10">
            <label class="label mr-1">App Title: </label><br />
            <input type="text" class="input" v-model="userObj.app_title" />
        </div>
    </div>
    <div class="w-10/10 flex flex-col md:flex-row-reverse mt-5 justify-center">
        <button class="btn btn-neutral w-10/10 md:w-5/10" @click="saveUser">
            save
        </button>
    </div>

</template>

<style scoped>

</style>
