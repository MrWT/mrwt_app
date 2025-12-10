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
        time: [],
        member: [],
    });
    // 提詞機 - 選擇
    let promptAction = ref("check");
    let promptTime = ref("All");
    let promptMember = ref("All");
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
        let fetchPromise_finMN = fetchData({
            api: "get_finance_month",
        }, "KUO-FUNDS");
        // 取得基金成員資訊
        let fetchPromise_members = fetchData({
            api: "get_members",
        }, "KUO-FUNDS");
        Promise.all([fetchPromise_aiRoles, fetchPromise_userInfo, fetchPromise_finMN, fetchPromise_members]).then((values) => {
            console.log("fetchInitData.values=", values);
            aiRoles = values[0];
            userInfo = values[1];

            // 帳務月份
            {
                promptOptions.time.push({ value: "All", text: "All", });
                values[2].forEach((fmn, fmn_i) => {
                    let f_fmn = fmn.split("-")[0] + " 年 " + fmn.split("-")[1] +  " 月 ";
                    promptOptions.time.push(
                        { value: f_fmn, text: f_fmn, }
                    );
                });
            }

            // 成員們
            {
                values[3].sort((x, y) => {
                    if(x["code_name"] < y["code_name"]) return -1;
                    if(x["code_name"] > y["code_name"]) return 1;
                    return 0;
                });

                promptOptions.member.push({ value: "All", text: "All", });
                values[3].forEach((memObj, mem_i) => {
                    promptOptions.member.push(
                        { value: memObj["name"], text: memObj["name"], }
                    );
                });
            }
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
    // 開啟 finance modal
    function openFinanceModal(){
        promptTime.value = "All";
        promptMember.value = "All";
        document.getElementById("financeModal").showModal();
    }
    // 組合提詞
    function combinePrompt(){

        let selTime = promptTime.value === "All" ? "全部時間" : promptTime.value;
        let selMember = promptMember.value === "All" ? "全部成員" : promptMember.value;

        prompt.value = "幫我列出 " + selTime + " " + selMember + " 的帳務紀錄(依時間倒序)";
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
        closeFinanceModal();
    }
    // 關閉 prompt modal
    function closeFinanceModal(){
        document.getElementById("financeModal").close();
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
        userMessage.value = "你好(以下對話都以台灣繁體中文回覆)";
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
        closeFinanceModal();
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
        <div class="tooltip tooltip-bottom md:tooltip-left" data-tip="帳務查詢">
            <button class="btn btn-circle bg-stone-500/70 hover:bg-blue-300" @click="openFinanceModal">
                <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 18h14M5 18v3h14v-3M5 18l1-9h12l1 9M16 6v3m-4-3v3m-2-6h8v3h-8V3Zm-1 9h.01v.01H9V12Zm3 0h.01v.01H12V12Zm3 0h.01v.01H15V12Zm-6 3h.01v.01H9V15Zm3 0h.01v.01H12V15Zm3 0h.01v.01H15V15Z"/>
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

<!-- finance modal -->
<dialog id="financeModal" class="modal">
    <div class="modal-box h-2/3 w-2/3 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span class="text-lg text-gray-900 text-center">帳務查詢</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="h-3/4 md:h-4/5 w-1/1 flex flex-col overflow-y-auto gap-2 border rounded-2xl">
            <div class="w-1/1 flex flex-row rounded-lg bg-state-200 p-2">
                <div class="flex-none pr-2">
                    時間:
                </div>
                <div class="flex-1">
                    <select v-model="promptTime" class="w-1/1 border">
                        <option v-for="(tObj, t_i) in promptOptions.time" :value="tObj.value">{{ tObj.text }}</option>
                    </select>
                </div>
            </div>
            <div class="w-1/1 flex flex-row rounded-lg bg-state-200 p-2">
                <div class="flex-none pr-2">
                    成員:
                </div>
                <div class="flex-1">
                    <select v-model="promptMember" class="w-1/1 border">
                        <option v-for="(mObj, m_i) in promptOptions.member" :value="mObj.value">{{ mObj.text }}</option>
                    </select>
                </div>
            </div>
        </div>

        <textarea class="textarea h-1/4 md:h-1/5 w-1/1 mt-1 bg-gray-900 text-gray-100" placeholder="想說點什麼嗎??" v-model="prompt" disabled></textarea>

        <div class="divider divider-primary"></div>
        <div class="modal-action">
            <button class="btn btn-ghost w-5/10 bg-gray-900 text-gray-100 hover:bg-yellow-100 hover:text-gray-900" @click.stop="closeFinanceModal">
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
