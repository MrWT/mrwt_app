<script setup>
    import { ref, reactive, onMounted, watch } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const emit = defineEmits(['popupMessage']);
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
    let chatBarStatus = ref(true);
    let chatState = ref("TALKING");
    // 回憶室 UUID
    let chat_room_uuid = ref("INIT");
    let messages = reactive([]);
    let userInfo = reactive({});
    let aiRoles = reactive([]);
    let currentAiRole = reactive({});
    // 提詞機 - 選項
    let promptOptions = reactive({
        scope: [
            { value: "temple-3year", text: "土城刈香", },
            { value: "temple-7bridge", text: "過年活動( 大廟七星橋 )", },
            { value: "temple-yearSafe", text: "過年活動( 大廟安太歲 )", },
            { value: "temple-foodmart", text: "過年活動( 大廟美食市集 )", },
            { value: "home-backMomHome", text: "過年活動( 回娘家 )", },
            { value: "home-29night", text: "過年活動( 除夕圍爐夜 )", },
            { value: "home-moon", text: "中秋烤肉活動", },
        ],
    });
    // 提詞機 - 選擇
    let promptScope = ref("home-moon");
    // 提詞機 - 結果
    let prompt = ref("");

    // 初始化 component
    function init(){
        console.log("chat.init");
        console.log("chat.props.title", props.title);
        console.log("chat.props.account", props.account);

        fetchInitData();
        chat("HI");

        combinePrompt();
    }
    // 取得 user 資料
    function fetchInitData(){
        // 取得 AI 角色
        let fetchAIRolesPromise = fetchData({
            api: "get_ai_role",
        }, "AI");
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

        chatState.value = "TALKING";
        let chatPromise = fetchData({
            api: "recall_memory",
            data: {
                account: props.account,
                chat_room_uuid: chat_room_uuid.value,
                message: message,
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

                    currentAiRole["speaker"] = speaker;
                    currentAiRole["short_name"] = short_name;
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

                chatState.value = "DONE";
            }, 100);
        });
    }
    // 送出 message
    function send(){
        // 當沒有 keyin message 時, 不送出訊息
        if(!userMessage.value) return;

        // 關閉對話 bar
        chatBarStatus.value = false;
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
    // 開啟 prompt modal
    function openPromptModal(){
        document.getElementById("promptModal").showModal();
    }
    // 組合提詞
    function combinePrompt(){
        let c_prompt = "";
        /*
        console.log("combinePrompt.promptScope=", promptScope.value);
        */

        if(promptScope.value){
            promptOptions.scope.forEach((scopeObj, act_i) => {
                if(promptScope.value === scopeObj.value){
                    c_prompt += scopeObj.text;
                }
            });
        }

        prompt.value = "我想聊聊 " + c_prompt;
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
    // 開關對話 bar
    function toggleChatBar(){
        chatBarStatus.value = !chatBarStatus.value;
    }

     // 監聽
    watch(promptScope, (newValue, oldValue) => {
        combinePrompt();
    });

</script>

<template>

<div id="chatBox" class="flex flex-col w-10/10 h-10/10">
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

<div class="join join-horizontal absolute left-0 z-10 w-1/1 justify-end px-2 gap-2"
     :class="{'bg-gray-200 bottom-12': chatBarStatus === true, 'bg-transparent bottom-2': chatBarStatus === false}">
    <!-- toggle 對話 bar -->
    <button class="btn join-item bg-gray-300 text-gray-900 font-black btn-circle border-0 border-black  hover:border-2" title="開關對話 bar" @click="toggleChatBar">
        <!-- open -->
        <svg v-if="chatBarStatus !== true" class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m5 15 7-7 7 7"/>
        </svg>

        <!-- close -->
        <svg v-if="chatBarStatus === true" class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m19 9-7 7-7-7"/>
        </svg>
    </button>
</div>
<div v-if="chatBarStatus === true" class="join join-horizontal absolute bottom-2 left-0 z-10 w-1/1 justify-center px-2 gap-2"
     :class="{'bg-gray-200': chatBarStatus === true, 'bg-transparent': chatBarStatus === false}">
    <input type="text" placeholder="想說點什麼呢?" class="input input-info join-item w-8/10" v-model="userMessage" @keyup.enter="send" />
    <button class="btn join-item bg-gray-300 btn-circle hover:bg-blue-300" @click="send">
        <svg class="size-4 text-gray-700 rotate-90" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
            <path fill-rule="evenodd" d="M12 2a1 1 0 0 1 .932.638l7 18a1 1 0 0 1-1.326 1.281L13 19.517V13a1 1 0 1 0-2 0v6.517l-5.606 2.402a1 1 0 0 1-1.326-1.281l7-18A1 1 0 0 1 12 2Z" clip-rule="evenodd"/>
        </svg>
    </button>
    <button class="btn join-item bg-gray-300 btn-circle hover:bg-blue-300" title="聊天提詞機" @click="openPromptModal">
        <svg class="size-4 text-gray-700" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 9h5m3 0h2M7 12h2m3 0h5M5 5h14a1 1 0 0 1 1 1v9a1 1 0 0 1-1 1h-6.616a1 1 0 0 0-.67.257l-2.88 2.592A.5.5 0 0 1 8 18.477V17a1 1 0 0 0-1-1H5a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1Z"/>
        </svg>
    </button>
</div>

<!-- prompt modal -->
<dialog id="promptModal" class="modal modal-end">
    <div class="modal-box h-10/10 max-w-10/10 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span class="text-lg text-gray-900 text-center">聊天提詞機</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="h-3/4 md:h-4/5 w-1/1 flex flex-col overflow-y-auto gap-2 border rounded-2xl">           
            <div class="h-1/3 w-1/1 flex flex-wrap rounded-lg bg-stone-200 px-2 gap-2 overflow-y-auto">
                <label v-for="(sObj, s_i) in promptOptions.scope" class="label text-gray-900">
                    <input type="radio" class="radio" :value="sObj.value" v-model="promptScope" />
                    {{ sObj.text }}
                </label>
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


</template>

<style scoped>

</style>
