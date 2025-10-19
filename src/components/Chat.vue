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
    // 開啟 setting modal
    function openSettingModal(){
        document.getElementById("settingModal").showModal();
    }
    // 關閉 setting modal
    function closeSettingModal(){
        document.getElementById("settingModal").close();
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

        document.getElementById("settingModal").close();
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

<div class="join join-horizontal absolute bottom-5 left-0 z-55 w-10/10 justify-center bg-gray-200">
    <input type="text" placeholder="想說點什麼呢?" class="input input-info join-item w-7/10" v-model="userMessage" @keyup.enter="send" />
    <button class="btn join-item bg-gray-700 btn-circle" @click="send">
        <svg class="size-5 text-white rotate-90" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
            <path fill-rule="evenodd" d="M12 2a1 1 0 0 1 .932.638l7 18a1 1 0 0 1-1.326 1.281L13 19.517V13a1 1 0 1 0-2 0v6.517l-5.606 2.402a1 1 0 0 1-1.326-1.281l7-18A1 1 0 0 1 12 2Z" clip-rule="evenodd"/>
        </svg>
    </button>
    <button class="btn join-item bg-gray-300 btn-circle" @click="openSettingModal">
        <svg class="size-5 text-gray-700" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
            <path fill-rule="evenodd" d="M9.586 2.586A2 2 0 0 1 11 2h2a2 2 0 0 1 2 2v.089l.473.196.063-.063a2.002 2.002 0 0 1 2.828 0l1.414 1.414a2 2 0 0 1 0 2.827l-.063.064.196.473H20a2 2 0 0 1 2 2v2a2 2 0 0 1-2 2h-.089l-.196.473.063.063a2.002 2.002 0 0 1 0 2.828l-1.414 1.414a2 2 0 0 1-2.828 0l-.063-.063-.473.196V20a2 2 0 0 1-2 2h-2a2 2 0 0 1-2-2v-.089l-.473-.196-.063.063a2.002 2.002 0 0 1-2.828 0l-1.414-1.414a2 2 0 0 1 0-2.827l.063-.064L4.089 15H4a2 2 0 0 1-2-2v-2a2 2 0 0 1 2-2h.09l.195-.473-.063-.063a2 2 0 0 1 0-2.828l1.414-1.414a2 2 0 0 1 2.827 0l.064.063L9 4.089V4a2 2 0 0 1 .586-1.414ZM8 12a4 4 0 1 1 8 0 4 4 0 0 1-8 0Z" clip-rule="evenodd"/>
        </svg>
    </button>
</div>

<!-- setting modal -->
<dialog id="settingModal" class="modal">
    <div class="modal-box h-8/10 w-8/10 flex flex-col bg-neutral-700">
        <div class="flex flex-col justify-center">
            <span class="text-xl text-white">AI 功能設定</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="h-8/10 w-10/10 flex flex-col overflow-y-auto">
            <div class="text-white">
                國家:
            </div>
            <div class="flex flex-row justify-end gap-2">            
                <label class="label text-white ">
                    <input type="radio" class="radio radio-primary" value="kr" v-model="currentAiRole.nation" />
                    韓國 
                </label>
                <label class="label text-white">
                    <input type="radio" class="radio radio-primary" value="jp" v-model="currentAiRole.nation" />
                    日本
                </label>
                <label class="label text-white">
                    <input type="radio" class="radio radio-primary" value="tw" v-model="currentAiRole.nation" />
                    台灣
                </label>
                <label class="label text-white">
                    <input type="radio" class="radio radio-primary" value="us" v-model="currentAiRole.nation" />
                    美國
                </label>
            </div>
            <div class="text-white">
                性別:
            </div>
            <div class="flex flex-row justify-start gap-2">
                <label class="label text-white">
                    <input type="radio" class="radio radio-secondary" value="boy" v-model="currentAiRole.gender" />
                    Boy
                </label>
                <label class="label text-white">
                    <input type="radio" class="radio radio-secondary" value="girl" v-model="currentAiRole.gender" />
                    Girl
                </label>
            </div>
        </div>
        <div class="modal-action">
            <button class="btn btn-ghost w-5/10 text-white" @click="closeSettingModal">
                關閉設定
            </button>

            <button class="btn btn-ghost w-5/10 text-white" @click="changeAiRole">
                儲存設定
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
