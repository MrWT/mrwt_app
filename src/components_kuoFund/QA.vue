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
    // 提詞機 - 選項
    let promptOptions = reactive({
        time: [
            { value: "0month", text: "這個月", },
            { value: "3month", text: "近三個月", },
            { value: "6month", text: "近半年", },
            { value: "1year", text: "近一年", },
            { value: "3year", text: "近三年", },
        ],
        member: [],
    });
    // 提詞機 - 選擇
    let promptAction = ref("check");
    let promptTime = ref("0month");
    let promptMember = ref("全部帳務");
    // 提詞機 - 結果
    let prompt = ref("");

    // 初始化 component
    function init(){
        console.log("QA.init");
        console.log("QA.props.title", props.title);
        console.log("QA.props.account", props.account);
        console.log("QA.props.user_role", props.user_role);

        fetchInitData();
        userMessage.value = "Hi";
        chat();

        // 先組合出預設 prompt
        combinePrompt();
    }
    // 取得初始資料
    function fetchInitData(){
        promptOptions.member.splice(0, promptOptions.member.length);

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
        // 取得基金成員資訊
        let fetchPromise_members = fetchData({
            api: "get_members",
        }, "KUO-FUNDS");
        Promise.all([fetchPromise_aiRoles, fetchPromise_userInfo, fetchPromise_members]).then((values) => {
            console.log("fetchInitData.values=", values);
            aiRoles = values[0];
            userInfo = values[1];

            promptOptions.member.push({ value: "全部帳務", text: "全部帳務", });
            
            values[2].forEach((memObj, mem_i) => {
                promptOptions.member.push(
                    { value: memObj["name"] + "帳務", text: memObj["name"] + "帳務", }
                );
            });
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
        }, "AI");
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
    // 開啟 prompt modal
    function openPromptModal(){
        document.getElementById("promptModal").showModal();
    }
    // 組合提詞
    function combinePrompt(){
        let c_prompt = "整理出";

        if(promptTime.value){
            promptOptions.time.forEach((timeObj, act_i) => {
                if(promptTime.value === timeObj.value){
                    let today = moment().format("YYYY-MM-DD");
                    let stDate = "";

                    switch(promptTime.value){
                        case "0month": stDate = moment().format("YYYY-MM"); break;
                        case "3month": stDate = moment().add(-3, "M").format("YYYY-MM"); break;
                        case "6month": stDate = moment().add(-6, "M").format("YYYY-MM"); break;
                        case "1year": stDate = moment().add(-12, "M").format("YYYY-MM"); break;
                        case "3year": stDate = moment().add(-36, "M").format("YYYY-MM"); break;
                    }
                    c_prompt += " " + stDate + "-01" + "~" + today + " ";
                }
            });
        }

        /*
        if(promptScope.value){
            promptOptions.scope.forEach((scopeObj, act_i) => {
                if(promptScope.value === scopeObj.value){
                    c_prompt += scopeObj.text;
                }
            });
        }
        */

        if(promptMember.value){
            if(promptOptions.member.length === 0){
                c_prompt += "全部帳務";
            }else{
                promptOptions.member.forEach((memObj, mem_i) => {
                    if(promptMember.value === memObj.value){
                        c_prompt += memObj.text;
                    }
                });
            }
        }
        
        prompt.value = c_prompt;
    }
    // 傳送提詞
    function sendPrompt(){
        if(prompt.value){
            chat_room_uuid.value = "INIT";
            userMessage.value = prompt.value;
            send();
        }else{
            let error_msg = "你必須提供點內容, 不然提詞機要怎麼幫你轉送~~";
            emit('popupMessage', false, error_msg); // Emitting the event with data
        }
        closePromptModal();
    }
    // 關閉 prompt modal
    function closePromptModal(){
        document.getElementById("promptModal").close();
    }
    // 開啟 NewChat 再確認 modal
    function openNewChatConfirmModal(){
        document.getElementById("newChatConfirmModal").showModal();
    }
    // 開新話題
    function newChat(){
        console.log("newChat");
        // 關閉再確認 modal
        closeNewChatConfirmModal();
        // 清空對話錄
        messages.splice(0, messages.length);
        chat_room_uuid.value = "INIT";
        // 開啟新對話
        userMessage.value = "Hi";
        chat();
    }
    // 關閉 NewChat 再確認 modal
    function closeNewChatConfirmModal(){
        document.getElementById("newChatConfirmModal").close();
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
        closeNewChatConfirmModal();
        closePromptModal();
    }

    // 監聽
    watch(promptAction, (newValue, oldValue) => {
        combinePrompt();
    });
    watch(promptMember, (newValue, oldValue) => {
        combinePrompt();
    });
    watch(promptTime, (newValue, oldValue) => {
        combinePrompt();
    });

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
        <div class="tooltip tooltip-bottom md:tooltip-left" data-tip="開啟新話題">
            <button class="btn btn-circle bg-red-300 text-gray-900 hover:bg-blue-300" @click="openNewChatConfirmModal">
                <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m15 9-6 6m0-6 6 6m6-3a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
                </svg>
            </button>
        </div>
        <div class="tooltip tooltip-bottom md:tooltip-left" data-tip="聊天提詞機">
            <button class="btn btn-circle bg-stone-500/70 hover:bg-blue-300" @click="openPromptModal">
                <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 9h5m3 0h2M7 12h2m3 0h5M5 5h14a1 1 0 0 1 1 1v9a1 1 0 0 1-1 1h-6.616a1 1 0 0 0-.67.257l-2.88 2.592A.5.5 0 0 1 8 18.477V17a1 1 0 0 0-1-1H5a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1Z"/>
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

<!-- prompt modal -->
<dialog id="promptModal" class="modal modal-end">
    <div class="modal-box h-10/10 max-w-10/10 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span class="text-lg text-gray-900 text-center">聊天提詞機</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="h-3/4 md:h-4/5 w-1/1 flex flex-col overflow-y-auto gap-2 border rounded-2xl">
            <div class="h-1/3 w-1/1 flex flex-row rounded-lg bg-state-200 px-2 gap-1 overflow-x-auto">
                <label v-for="(tObj, t_i) in promptOptions.time" class="label text-gray-900">
                    <input type="radio" class="radio" :value="tObj.value" v-model="promptTime" />
                    {{ tObj.text }}
                </label>
            </div>
            <div class="h-1/3 w-1/1 flex flex-row rounded-lg bg-state-200 px-2 gap-1 overflow-x-auto">
                <select v-model="promptMember" class="w-1/1">
                    <option v-for="(mObj, m_i) in promptOptions.member" :value="mObj.value">{{ mObj.text }}</option>
                </select>
            </div>
        </div>

        <textarea class="textarea h-1/4 md:h-1/5 w-1/1 mt-1 bg-gray-900 text-gray-100" placeholder="想說點什麼嗎??" v-model="prompt"></textarea>

        <div class="divider divider-primary"></div>
        <div class="modal-action">
            <button class="btn btn-ghost w-5/10 bg-gray-200 text-gray-900 hover:bg-yellow-100" @click.stop="closePromptModal">
                關閉
            </button>

            <button class="btn btn-ghost w-5/10 bg-gray-200 text-gray-900 hover:bg-yellow-100" @click.stop="sendPrompt">
                傳送
            </button>
        </div>
    </div>
    <form method="dialog" class="modal-backdrop">
        <button>close</button>
    </form>
</dialog>

<!-- newChatConfirm modal -->
<dialog id="newChatConfirmModal" class="modal">
    <div class="modal-box h-3/10 w-10/10 flex flex-col bg-neutral-500">
        <div class="h-10/10 w-10/10 text-center text-black font-black">
            <span class="text-2xl">再跟您確認一次~</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 text-gray-900 bg-gray-100 hover:bg-gray-500 hover:text-gray-100 hover:underline" @click="closeNewChatConfirmModal">
                話題繼續
            </button>

            <button class="btn btn-ghost w-1/2 text-gray-900 bg-gray-100 hover:bg-gray-500 hover:text-gray-100 hover:underline" @click="newChat">
                新話題
            </button>
        </div>
    </div>
    <form method="dialog" class="modal-backdrop">
        <button>close</button>
    </form>
</dialog>


</template>

<style scoped>

</style>
