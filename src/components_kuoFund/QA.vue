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
    let chatModalStatus = ref("CLOSE");
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
        // 關閉全部 modal
        closeAllModal();
        
        chatState.value = "TALKING";
        let chatPromise = fetchData({
            api: "chat",
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

            messages.push({
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
                chatBoxElement.scrollTop = chatBoxElement.scrollHeight;
            });
        });

    }
    // 送出 message
    function send(){
        // 當沒有 keyin message 時, 不送出訊息
        if(!userMessage.value) return;

        let user_name = userInfo.language === "EN" ? userInfo.name : userInfo.cname;

        messages.push({
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

            // 關閉全部 modal
            closeAllModal();
        });
    }
    // 開啟 chat modal
    function openChatModal(){
        chatModalStatus.value = "OPEN";
        document.getElementById("chatModal").showModal();
    }
    // 關閉 chat modal
    function closeChatModal(){
        chatModalStatus.value = "CLOSE";
        document.getElementById("chatModal").close();
    }
    // 關閉全部 modal
    function closeAllModal(){
        closeChatModal();
    }

</script>

<template>

<div class="w-1/1 h-1/1 flex flex-col md:flex-row-reverse">

    <!-- function button bar -->
    <div class="w-1/1 md:w-1/12 md:h-1/1 flex flex-row md:flex-col justify-center items-end shadow-2xl">
        <div class="tooltip tooltip-bottom md:tooltip-left" data-tip="說點什麼">
            <button class="btn btn-circle bg-green-300 text-gray-900 hover:bg-blue-300" @click="openChatModal">
                <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17h6l3 3v-3h2V9h-2M4 4h11v8H9l-3 3v-3H4V4Z"/>
                </svg>
            </button>
        </div>
    </div>

    <!-- 聊天內容 -->
    <div id="chatBox" class="flex flex-col w-1/1 h-11/12 md:h-1/1 md:w-11/12 mt-15 md:mt-5 overflow-y-auto">
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

<!-- chat modal -->
<dialog id="chatModal" class="modal modal-end">
    <div class="modal-box h-8/10 w-1/1 md:w-8/10 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span class="text-xl text-gray-900 text-center">想說點什麼呢?</span>
        </div>
        <div class="h-8/10 w-10/10 flex flex-col overflow-y-auto gap-2">
            <div class="divider divider-primary"></div>
            <div class="w-1/1 h-1/1 flex flex-col">
                <textarea class="textarea w-1/1 h-1/1" v-model="userMessage" @keyup.ctrl.enter="send"></textarea>
                <span class="w-1/1 text-xs text-gray-900/60 text-center hidden sm:block">( enter: 換行 / ctrl + enter: 直接傳送訊息 )</span>
            </div>
        </div>
        <div class="divider divider-primary"></div>
        <div class="modal-action">
            <button class="btn btn-ghost bg-gray-900 text-gray-100 hover:bg-gray-100 hover:text-gray-900 w-1/2" @click="closeChatModal">
                關閉
            </button>
            <button class="btn bg-gray-300 hover:bg-blue-300 w-1/2" @click="send">
                <svg class="size-4 text-gray-700 rotate-90" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                    <path fill-rule="evenodd" d="M12 2a1 1 0 0 1 .932.638l7 18a1 1 0 0 1-1.326 1.281L13 19.517V13a1 1 0 1 0-2 0v6.517l-5.606 2.402a1 1 0 0 1-1.326-1.281l7-18A1 1 0 0 1 12 2Z" clip-rule="evenodd"/>
                </svg>
                傳給 AI
            </button>
        </div>
    </div>
</dialog>

</template>

<style scoped>

</style>
