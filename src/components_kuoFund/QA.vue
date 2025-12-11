<script setup>
    import { ref, reactive, onMounted, onUpdated, watch, nextTick } from 'vue'
    import moment from 'moment'
    import { gsap } from "gsap"
    import { fetchData } from "@/composables/fetchData"

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        user_role: String,
    })

    onMounted(() => {
        console.log("QA mounted.");
        init();
    });

    onUpdated(() => {
        console.log("QA updated.");

    });

    let appState = ref("");
    let userMessage = ref("");
    let chatState = ref("TALKING");
    // 聊天室 UUID
    let chat_room_uuid = ref("INIT");
    let messages = reactive([]);
    let userInfo = reactive({});
    let aiRoles = reactive([]);
    let currentAiRole = reactive({});

    // 初始化 component
    function init(){
        console.log("QA.init");
        console.log("QA.props.title", props.title);
        console.log("QA.props.account", props.account);
        console.log("QA.props.user_role", props.user_role);

        fetchInitData();
        userMessage.value = "Hi";
        chat();
    }
    // 取得初始資料
    function fetchInitData(){
        // 取得 AI 角色
        let fetchPromise_aiRoles = fetchData({
            api: "get_ai_role",
        }, "AI");
        // 取得使用者資訊
        let fetchPromise_userInfo = fetchData({
            api: "get_user",
            data: {
                account: props.account,
            }
        });
        Promise.all([fetchPromise_aiRoles, fetchPromise_userInfo]).then((values) => {
            console.log("fetchInitData.values=", values);
            aiRoles = values[0];
            userInfo = values[1];
        });
    }
    // chat with ai
    function chat(){
        console.log("chat.message=" + userMessage.value);
        
        chatState.value = "TALKING";
        let chatPromise = fetchData({
            api: "qa",
            data: {
                account: props.account,
                chat_room_uuid: chat_room_uuid.value,
                message: userMessage.value,
                time: moment().format("YYYY-MM-DD HH:mm:ss"),
            }
        }, "KUO-FUNDS");
        Promise.all([chatPromise]).then((values) => {
            console.log("chatPromise.values=", values);

            chat_room_uuid.value = values[0]["chat_room_uuid"];
            let ai_msg = values[0]["message"];
            let speaker = "";
            let short_name = "";
            aiRoles.forEach((roleObj, role_i) => {
                if(roleObj["code"] === values[0]["ai_role"]){
                    speaker = roleObj["name"];
                    short_name = roleObj["short_name"];

                    currentAiRole["speaker"] = roleObj["speaker"];
                    currentAiRole["short_name"] = roleObj["short_name"];
                    currentAiRole["nation"] = roleObj["nation"];
                    currentAiRole["gender"] = roleObj["gender"];
                }
            });

            messages.unshift({
                role: "AI",
                speaker: speaker,
                short_name: short_name,
                message: ai_msg,
                time: moment().format("HH:mm:ss"),
            });

            // Vue3 因資料改變 DOM 後觸發
            nextTick(() => {
                chatState.value = "DONE";
                userMessage.value = "";

                // 讓 app scroll 到底
                let chatBoxElement = document.getElementById("chatBox");
                chatBoxElement.scrollTop = 0;// chatBoxElement.scrollHeight;
            });
        });

    }
    // 送出 message
    function send(){
        console.log("send.userMessage.value=" + userMessage.value);

        // 當沒有 keyin message 時, 不送出訊息
        if(!userMessage.value) return;

        let user_name = userInfo.language === "EN" ? userInfo.name : userInfo.cname;

        messages.unshift({
            role: "user",
            speaker: user_name,
            short_name: props.account.substr(0, 1),
            message: userMessage.value,
            time: moment().format("HH:mm:ss"),
        });

        chat();
    }      
    // 傳送至 Line
    function sendToLine(){
        let linePromise = fetchData({
            api: "send_to_line",
            data: {
                account: props.account,
                messages: userMessage.value,
            }
        });
        Promise.all([linePromise]).then((values) => {
            console.log("linePromise.values=", values);
            userMessage.value = "";

        });
    }

</script>

<template>

<div class="w-1/1 h-1/1 flex flex-col md:flex-row-reverse">

    <!-- function button bar -->
    <div class="w-1/1 shadow-2xl flex flex-row">
        <div class="flex-1">
            <textarea class="textarea w-1/1 h-1/1 md:h-100" v-model="userMessage" placeholder="想說點什麼呢?" :disabled="chatState === 'TALKING'"></textarea>
        </div>
        <div class="flex-none">
            <button class="btn bg-gray-300 hover:bg-blue-300 rounded-xl w-1/1 h-1/1" @click="send">
                <svg v-if="chatState !== 'TALKING'" class="size-4 text-gray-700 rotate-90" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                    <path fill-rule="evenodd" d="M12 2a1 1 0 0 1 .932.638l7 18a1 1 0 0 1-1.326 1.281L13 19.517V13a1 1 0 1 0-2 0v6.517l-5.606 2.402a1 1 0 0 1-1.326-1.281l7-18A1 1 0 0 1 12 2Z" clip-rule="evenodd"/>
                </svg>
                <span v-if="chatState !== 'TALKING'">傳送</span>
                <span v-if="chatState === 'TALKING'" class="loading loading-spinner loading-md"></span>
            </button>
        </div>
    </div>

    <!-- 聊天內容 -->
    <div id="chatBox" class="flex flex-col w-1/1 h-11/12 md:h-1/1 md:w-11/12 mt-5 overflow-y-auto">
        <div v-for="(msgObj, msg_i) in messages" class="chat"
            :class="{ 'chat-start': msgObj.role === 'AI', 'chat-end': msgObj.role === 'user' }">
            <div class="chat-image avatar">
                <div class="avatar avatar-placeholder">
                    <div class="w-8 rounded-full border-5 bg-white text-gray-900"
                        :class="{'border-zinc-500': msgObj.role === 'AI', 'border-stone-500': msgObj.role === 'user'}">
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
        <div v-if="chatState === 'TALKING'" class="chat chat-start">
            <div class="chat-image avatar">
                <div class="avatar avatar-placeholder">
                    <div class="size-8 rounded-full bg-neutral text-gray-100">
                        <span class="text-xs">
                            {{ " " }}
                        </span>
                    </div>
                </div>
            </div>
            <div class="chat-header">
                {{ "AI" }}
            </div>
            <div class="chat-bubble">
                好喔~ 稍等
                <span class="loading loading-dots loading-xs"></span>
            </div>
        </div>
    </div>
</div>

</template>

<style scoped>

</style>
