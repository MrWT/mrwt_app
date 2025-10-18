<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("Chat mounted.");
        init();
    });

    let appState = ref("");
    let userMessage = ref("");
    let messages = reactive([]);
    let userInfo = reactive({});
    let aiRoles = reactive([]);
    let currentAiRole = reactive({});

    // 初始化 component
    function init(){
        console.log("chat.init");
        console.log("chat.props.title", props.title);
        console.log("chat.props.account", props.account);

        fetchInitData();
        chat("INIT");
    }
    // 取得初始資料
    function fetchInitData(){
        // 取得 AI 角色
        let fetchAIRolesPromise = fetchData({
            api: "get_ai_role",
        });
        // 取得使用者資訊
        let fetchUserInfoPromise = fetchData({
            api: "get_user",
            data: {
                account: props.account,
            }
        });
        Promise.all([fetchAIRolesPromise, fetchUserInfoPromise]).then((values) => {
            console.log("fetchInitData.values=", values);
            aiRoles = values[0];
            userInfo = values[1];
        });
    }
    // chat with ai
    function chat(message){
        console.log("chat.message=" + message);

        let chatPromise = fetchData({
            api: "chat",
            data: {
                account: props.account,
                message: message,
                time: moment().format("YYYY-MM-DD HH:mm:ss"),
            }
        });
        Promise.all([chatPromise]).then((values) => {
            console.log("chatPromise.values=", values);

            let ai_msg = values[0]["message"];
            let speaker = "";
            let short_name = "";
            aiRoles.forEach((roleObj, role_i) => {
                if(roleObj["role"] === values[0]["ai_role"]){
                    speaker = roleObj["name"];
                    short_name = roleObj["name"].substr(0, 1);

                    currentAiRole["nation"] = roleObj["role"].split("_")[0];
                    currentAiRole["gender"] = roleObj["role"].split("_")[1];
                }
            });

            messages.push({
                role: "AI",
                speaker: speaker,
                short_name: short_name,
                message: ai_msg,
                time: moment().format("HH:mm:ss"),
            });

            setTimeout(() => {
                let chatBoxElement = document.getElementById("chatBox");
                chatBoxElement.scrollTo(0, chatBoxElement.scrollHeight);
            }, 100);
        });

    }
    // 送出 message
    function send(){
        let user_name = userInfo.language === "EN" ? userInfo.name : userInfo.cname;

        messages.push({
            role: "user",
            speaker: user_name,
            short_name: props.account.substr(0, 1),
            message: userMessage.value,
            time: moment().format("HH:mm:ss"),
        });

        chat(userMessage.value);

        setTimeout(() => {
            userMessage.value = "";
            let chatBoxElement = document.getElementById("chatBox");
            chatBoxElement.scrollTo(0, chatBoxElement.scrollHeight);
        }, 100);
    }        
    // 改變 AI 角色
    function changeAiRole(){
        console.log("changeAiRole.currentAiRole=", currentAiRole);
        let msg = "換成";
        switch(currentAiRole.nation){
            case "kr":
                msg += "韓國";
                break;
            case "jp":
                msg += "日本";
                break;
            case "tw":
                msg += "台灣";
                break;
            case "us":
                msg += "美國";
                break;
        }
        switch(currentAiRole.gender){
            case "boy":
                msg += "男生";
                break;
            case "girl":
                msg += "女生";
                break;
        }
        chat(msg);
    }
        

</script>

<template>

<div id="chatBox" class="flex flex-col w-10/10 h-8/10 overflow-y-auto">
    <div v-for="(msgObj, msg_i) in messages" class="chat"
        :class="{ 'chat-start': msgObj.role === 'AI', 'chat-end': msgObj.role === 'user' }">
        <div class="chat-image avatar">
            <div class="avatar avatar-placeholder">
                <div class="w-8 rounded-full"
                    :class="{'bg-neutral': msgObj.role === 'AI', 'text-gray-100': msgObj.role === 'AI',
                             'bg-green-500': msgObj.role === 'user', 'text-gray-900': msgObj.role === 'user'}">
                    <span class="text-xs">{{ msgObj.short_name }}</span>
                </div>
            </div>
        </div>
        <div class="chat-header">
            {{ msgObj.speaker }}
            <time class="text-xs opacity-50">{{ msgObj.time }}</time>
        </div>
        <div class="chat-bubble">
            <p style="white-space:pre-wrap;">
                {{ msgObj.message }}
            </p>
        </div>
    </div>
</div>

<div class="join join-horizontal absolute bottom-15 left-0 z-55 w-10/10 justify-center bg-gray-200">
    <input type="text" placeholder="想說點什麼呢?" class="input input-info join-item w-7/10" v-model="userMessage" @keyup.enter="send" />
    <button class="btn join-item bg-gray-700 btn-circle" @click="send">
        <svg class="size-6 text-white rotate-90" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
            <path fill-rule="evenodd" d="M12 2a1 1 0 0 1 .932.638l7 18a1 1 0 0 1-1.326 1.281L13 19.517V13a1 1 0 1 0-2 0v6.517l-5.606 2.402a1 1 0 0 1-1.326-1.281l7-18A1 1 0 0 1 12 2Z" clip-rule="evenodd"/>
        </svg>
    </button>
</div>
<div class="absolute bottom-5 left-0 z-55 w-10/10 flex flex-row justify-center gap-10">
    <div class="flex flex-row justify-end gap-2">
        <label class="label">
            <input type="radio" class="radio radio-primary" value="kr" v-model="currentAiRole.nation" @change="changeAiRole" />
            韓國 
        </label>
        <label class="label">
            <input type="radio" class="radio radio-primary" value="jp" v-model="currentAiRole.nation" @change="changeAiRole" />
            日本
        </label>
        <label class="label">
            <input type="radio" class="radio radio-primary" value="tw" v-model="currentAiRole.nation" @change="changeAiRole" />
            台灣
        </label>
        <label class="label">
            <input type="radio" class="radio radio-primary" value="us" v-model="currentAiRole.nation" @change="changeAiRole" />
            美國
        </label>
    </div>
    <div class="flex flex-row justify-start gap-2">
        <label class="label">
            <input type="radio" class="radio radio-secondary" value="boy" v-model="currentAiRole.gender" @change="changeAiRole" />
            男生
        </label>
        <label class="label">
            <input type="radio" class="radio radio-secondary" value="girl" v-model="currentAiRole.gender" @change="changeAiRole" />
            女生
        </label>
    </div>
</div>

</template>

<style scoped>

</style>
