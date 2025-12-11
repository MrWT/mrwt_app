<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import { gsap } from "gsap"
    import { fetchData } from "./composables/fetchData"
    import { getRandomNumber } from "./composables/random"

    import Finance from './components/Finance.vue'
    import Gallery from './components/Gallery.vue'
    import Footmark from './components/Footmark.vue'
    import Readme from './components/Readme.vue'
    import Setting from './components/Setting.vue'
    import Chat from './components/Chat.vue'
    import Imagen from './components/Imagen.vue'
    import PlanTrip from './components/PlanTrip.vue'
    import TripSchedule from './components/TripSchedule.vue'
    import Author from './components/Author.vue'
    import Recollection from './components/Recollection.vue'
    import Achievement from './components/Achievement.vue'
    import PopupMessage from './components/PopupMessage.vue'

    // 郭家基金相關 component - 先寄生於此專案, 待成熟後, 再分家出去
    import QA from './components_kuoFund/QA.vue'
    import Survey from './components_kuoFund/Survey.vue'
    import Finance_KF from './components_kuoFund/Finance.vue'
    import Rule_KF from './components_kuoFund/Rule.vue'
    import Activity_KF from './components_kuoFund/Activity.vue'

    onMounted(() => {
        console.log("App mounted.");
        init();

        // 沒有使用者資訊時, 跳出 signinModal
        if(!userInfo.account){
            document.getElementById("signinModal").showModal();
            setTimeout(() => {
                if(!isAutoSignin.value){
                    document.getElementById("signinName").focus();
                    genSigninAnimation_gsap();
                }
            }, 400);

        }
    });

    // 系統資訊
    let appState = ref("");
    // 一般 - 功能設定
    let appSetting = reactive({
        state: "",
        contentComponent: "gallery",
        title: "",
        googleMapApiKey: "",
        reference: [],
        funButtons: [],
        imagenOption: {},
    });
    // 郭家基金 - 功能設定
    let kf_funSetting = reactive({
        bank_info: {},
        file_info: {},
        member: {},
        activity_annouce_period: {
            start: "",
            end: "",
        },
        agent: {
            name: "",
            phone: "",
            lineID: "",
            term:"",
        },
    });
    // 使用者資訊
    let isAutoSignin = ref(false);
    let signinError = ref(false);
    const tempAccount = ref("");
    const userInfo = reactive({
        account: null,
        name: null,
        shortName: null,
        mail: null,
        role: null,
        funcs: [],
        languages: {},
        focus_news_topic: null,
    });
    // Popup Message
    let opObj = reactive({
        status: false,
        message: "",
    });

    // 初始化 app
    function init(){
        // 自動登入控制
        {
            const urlParams = new URLSearchParams(window.location.search);
            const user = urlParams.get('user'); // 取得 user

            if(user && user.trim()){
                isAutoSignin.value = true;

                // get_all_account
                let fetchAllAccount = fetchData({
                    api: "get_all_account",
                });

                Promise.all([fetchAllAccount]).then((values) => {
                    console.log("get_all_account.values=", values);

                    values[0].forEach((account, account_i) => {
                        if(account.toUpperCase() === user.trim().toUpperCase()){
                            tempAccount.value = account.toUpperCase();
                            signin();
                        }
                    });
                });
            }
        }
    }
    // 重設系統設定
    function resetAppSetting(){
        appSetting.contentComponent = "gallery";
        appSetting.title = "V.Demo";
    }
    // 置換 component
    function gotoPage(page) {
        // close userInfoModal
        document.getElementById("userInfoModal").close();

        appSetting.contentComponent = "reset";
        setTimeout(() => {
            appSetting.contentComponent = page;
        }, 200);
    }
    // keyin account
    function keyinAccount(event){
        tempAccount.value = event.target.value.toUpperCase();
    }
    // 重設使用者資訊
    function resetUserInfo(){
        tempAccount.value = "";
        userInfo.account = null;
        userInfo.name = null;
        userInfo.shortName = null;
        userInfo.mail = null;
        userInfo.role = null;
        userInfo.funcs = [];
        userInfo.languages = {};
    }
    // 登入
    function signin(){
        //console.log("signin.click");
        //console.log("tempAccount.value=", tempAccount.value);

        if(tempAccount.value){
            // close signinModal
            document.getElementById("signinModal").close();

            let fetchAppSetting = fetchData({
                api: "get_app_setting",
            });

            let fetchUserData = fetchData({
                api: "login",
                data: {
                    account: tempAccount.value.trim()
                }
            });

            let fetchUserLanguage = fetchData({
                api: "get_language",
                data: {
                    account: tempAccount.value.trim()
                }
            });

            Promise.all([fetchAppSetting, fetchUserData, fetchUserLanguage]).then((values) => {
                console.log("signin.values=", values);

                if(!values || !values[1]){
                    signinError.value = true;
                    signout();
                    return;
                }
                signinError.value = false;

                let appSettingObj = values[0];
                console.log("appSettingObj=", appSettingObj);

                // appSetting.googleMapApiKey
                appSetting.googleMapApiKey = appSettingObj["google_map_api_key"];

                // 郭家基金 - 功能設定值
                kf_funSetting.bank_info = appSettingObj["kf_bank_info"];
                kf_funSetting.file_info = appSettingObj["kf_file_info"];
                kf_funSetting.member = appSettingObj["kf_member"];
                kf_funSetting.activity_annouce_period.start = appSettingObj["kf_activity_annouce_period"]["value1"];
                kf_funSetting.activity_annouce_period.end = appSettingObj["kf_activity_annouce_period"]["value2"];
                kf_funSetting.agent.name = appSettingObj["agent_name"];
                kf_funSetting.agent.phone = appSettingObj["agent_phone"];
                kf_funSetting.agent.lineID = appSettingObj["agent_line_id"];
                kf_funSetting.agent.term = appSettingObj["agent_term"];

                // appSetting.reference
                {
                    appSetting.reference = [];
                    appSettingObj["reference"].sort((x, y) => {
                        return x["seq"] - y["seq"];
                    });
                    appSettingObj["reference"].forEach((appSetObj, as_i) => {
                        appSetting.reference.push({
                            index: ((as_i + 1) < 10 ? "0" : "") + (as_i + 1),
                            name: appSetObj["key"],
                            text: appSetObj["display_text"],
                            link: appSetObj["link_url"],
                            arch: appSetObj["architecture"],
                        });
                    });
                }

                // user info
                let userInfoObj = values[1];
                {
                    console.log("userInfoObj=", userInfoObj);
                    userInfo.account = userInfoObj["account"];
                    userInfo.name = userInfoObj["name"];
                    userInfo.cname = userInfoObj["cname"];
                    userInfo.shortName = userInfo.name.substr(0, 1);
                    userInfo.mail = userInfoObj["mail"];
                    userInfo.role = userInfoObj["role"];
                    userInfo.funcs = userInfoObj["function"];
                    userInfo.focus_news_topic = userInfoObj["focus_news_topic"];
                    appSetting.title = userInfoObj["app_title"];
                }

                // user language
                {
                    let userLangObj = values[2];
                    console.log("userLangObj=", userLangObj);
                    userInfo.languages = {};
                    userLangObj["words"].forEach((word, w_i) => {
                        let w_key = word["name"];
                        let w_display = word["display_text"];

                        userInfo.languages[w_key] = w_display;
                    });
                }

                // 功能 buttons
                {
                    appSetting.funButtons.splice(0, appSetting.funButtons.length);

                    let allFunctionKeys = ["finance", "chat", "imagen", "plan_trip", "trip_schedule", "footmark", "recollection", "achievement", "qa", "finance_kf", "rule_kf", "activity_kf"];
                    let buildingFunctionKeys = [];
                    let buildingFunctionKeys_kf = [];
                    allFunctionKeys.forEach((funKey, fk_i) => {
                        if(userInfo.funcs.indexOf(funKey) >= 0){

                            if(buildingFunctionKeys.indexOf(funKey) >= 0 && userInfo.role === "admin"){
                                appSetting.funButtons.push({ key: funKey, display_text: userInfo.languages[funKey] });
                            }                            
                            if(buildingFunctionKeys_kf.indexOf(funKey) >= 0 && userInfo.role === "admin_kf"){
                               appSetting.funButtons.push({ key: funKey, display_text: userInfo.languages[funKey] });
                            }
                            if(buildingFunctionKeys.indexOf(funKey) < 0 && buildingFunctionKeys_kf.indexOf(funKey) < 0){
                               appSetting.funButtons.push({ key: funKey, display_text: userInfo.languages[funKey] });
                            }
                        }
                    });
                }

                // close signinModal
                document.getElementById("signinModal").close();

                if(userInfoObj["account"].toUpperCase() === "KUOFAMILY" || userInfoObj["role"] === "admin_kf"){
                    // 郭家基金成員 - 預設開啟基金明細 component
                    gotoPage("finance_kf");
                } else if(userInfoObj["account"].toUpperCase().startsWith("SURVEY")){
                    // 郭家基金-問卷帳號 - 預設開啟問卷 component
                    gotoPage("survey");
                }else{
                    // 預設開啟 gallery component
                    gotoPage('gallery');
                }
            });
        }
    }
    // 登出
    function signout(){
        // close userInfoModal
        document.getElementById("userInfoModal").close();

        resetUserInfo();
        resetAppSetting();
        document.getElementById("signinModal").showModal();
        setTimeout(() => {
            document.getElementById("signinName").focus();
        }, 400);
    }
    // 前往作者簡介
    function gotoIntroduceAuthor(data){
        console.log("gotoIntroduceAuthor.data=", data);
        gotoPage("author");
    }
    // 使用者資訊
    function openUserInfoModal(){
        document.getElementById("userInfoModal").showModal();
    }
    // 產生登入 modal 動畫
    function genSigninAnimation_gsap(){
        let signinModal = document.getElementById("signinModal");
        console.log("signinModal.clientHeight=", signinModal.clientHeight);
        console.log("signinModal.clientWidth=", signinModal.clientWidth);
        
        // 碎花片 - 候選顏色
        let boxColors = ["green", "blue", "purple", "gold", "peru", "blanchedalmond", "blueviolet", "goldenrod"];
        // 碎花片 - 數量上限
        let boxCount = 500;

        let boxContainer = document.getElementById("boxContainer");
        for(let box_i = 1; box_i <= boxCount; box_i++){
            let box_div = document.createElement("div");
            box_div.setAttribute("id", "box_" + box_i);
            let boxColorIndex = getRandomNumber(0, boxColors.length - 1);
            box_div.style.backgroundColor = boxColors[ boxColorIndex ];
            box_div.style.width = "30px";
            box_div.style.height = "30px";

            boxContainer.append(box_div);
        }

        // 控制 gsap animation
        let tlList = [];
        for(let box_i = 1; box_i <= boxCount; box_i++){
            let uniqueId = "" + box_i;

            let tl = gsap.timeline({ yoyo: true, repeat: -1 });
            tl.to("#" + "box_" + uniqueId, {
                id: "tween_" + uniqueId,
                duration: getRandomNumber(10, 20)/10,
                opacity: 0,
            });
            tlList.push(tl);
        }
        
    }
    // popup message
    function popupMessage(status, message){
        opObj.status = status;
        opObj.message = message;

        document.getElementById("alertMsg").classList.remove("hidden");

        setTimeout(() => {
            document.getElementById("alertMsg").classList.add("hidden");
        }, 5000);
    }

</script>

<template>
    <!-- 系統 loading mask -->
    <div id="loading" class="w-full h-full flex justify-center items-center bg-gray-700/70 absolute top-0 left-0 z-[9999]" style="display:none;">
        <span class="loading loading-bars loading-xl"></span>
    </div>
    <!-- 功能 component -->
    <div class="navbar bg-yellow-800/50 h-[10px] fixed top-0 left-0 z-50">
        <div class="flex-1">
            <a class="btn btn-ghost text-xl" @click="gotoPage('gallery')">
                <span>{{ appSetting.title }}</span>
            </a>
        </div>
        <div class="flex-none">
            <ul class="menu menu-horizontal bg-slate-200/100 rounded-box w-10/10">
                <!-- readme -->
                <li @click="gotoPage('readme')">
                    <a class="tooltip tooltip-bottom" :data-tip="userInfo.languages.readme">
                        <svg class="size-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 11h2v5m-2 0h4m-2.592-8.5h.01M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
                        </svg>
                    </a>
                </li>
                <!-- setting -->
                <li v-if="userInfo.funcs.indexOf('setting') !== -1" @click="gotoPage('setting')">
                    <a class="tooltip tooltip-bottom" :data-tip="userInfo.languages.app_setting">
                        <svg class="size-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                            <path fill-rule="evenodd" d="M9.586 2.586A2 2 0 0 1 11 2h2a2 2 0 0 1 2 2v.089l.473.196.063-.063a2.002 2.002 0 0 1 2.828 0l1.414 1.414a2 2 0 0 1 0 2.827l-.063.064.196.473H20a2 2 0 0 1 2 2v2a2 2 0 0 1-2 2h-.089l-.196.473.063.063a2.002 2.002 0 0 1 0 2.828l-1.414 1.414a2 2 0 0 1-2.828 0l-.063-.063-.473.196V20a2 2 0 0 1-2 2h-2a2 2 0 0 1-2-2v-.089l-.473-.196-.063.063a2.002 2.002 0 0 1-2.828 0l-1.414-1.414a2 2 0 0 1 0-2.827l.063-.064L4.089 15H4a2 2 0 0 1-2-2v-2a2 2 0 0 1 2-2h.09l.195-.473-.063-.063a2 2 0 0 1 0-2.828l1.414-1.414a2 2 0 0 1 2.827 0l.064.063L9 4.089V4a2 2 0 0 1 .586-1.414ZM8 12a4 4 0 1 1 8 0 4 4 0 0 1-8 0Z" clip-rule="evenodd"/>
                        </svg>
                    </a>
                </li>
                <!-- userInfo -->
                <li @click="openUserInfoModal">
                    <a class="tooltip tooltip-bottom" :data-tip="userInfo.languages.user_setting">
                        <svg class="size-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                            <path fill-rule="evenodd" d="M4 4a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2H4Zm10 5a1 1 0 0 1 1-1h3a1 1 0 1 1 0 2h-3a1 1 0 0 1-1-1Zm0 3a1 1 0 0 1 1-1h3a1 1 0 1 1 0 2h-3a1 1 0 0 1-1-1Zm0 3a1 1 0 0 1 1-1h3a1 1 0 1 1 0 2h-3a1 1 0 0 1-1-1Zm-8-5a3 3 0 1 1 6 0 3 3 0 0 1-6 0Zm1.942 4a3 3 0 0 0-2.847 2.051l-.044.133-.004.012c-.042.126-.055.167-.042.195.006.013.02.023.038.039.032.025.08.064.146.155A1 1 0 0 0 6 17h6a1 1 0 0 0 .811-.415.713.713 0 0 1 .146-.155c.019-.016.031-.026.038-.04.014-.027 0-.068-.042-.194l-.004-.012-.044-.133A3 3 0 0 0 10.059 14H7.942Z" clip-rule="evenodd"/>
                        </svg>
                    </a>
                </li>
            </ul>
        </div>
    </div>
    <!-- 為了製造出"功能 menu 白霧底效果" -->
    <div class="navbar bg-white opacity-80 h-[10px] fixed top-15 left-0 z-50">
    </div>
    <!-- 功能 menu -->
    <div class="navbar shadow-lg h-[10px] fixed top-15 left-0 z-50 flex flex-row justify-start content-center gap-5 overflow-x-auto overflow-y-hidden">
        <button v-for="(fbObj, fb_i) in appSetting.funButtons" class="btn btn-soft btn-primary text-black font-black rounded-md hover:bg-violet-200/30" 
                :class="{'bg-violet-200/50':appSetting.contentComponent === fbObj.key, 'btn-ghost': appSetting.contentComponent !== fbObj.key}" 
                @click="gotoPage(fbObj.key)">
            {{ fbObj.display_text }}
        </button>
    </div>
    <!-- 功能 component -->
    <div class="p-4 h-8/10 mt-30">
        <Gallery v-if="appSetting.contentComponent === 'gallery'" :title="appSetting.title" :account="userInfo.account" :cname="userInfo.cname" :user_role="userInfo.role" :focus_news_topic="userInfo.focus_news_topic" @popup-message="popupMessage" />
        <Readme v-else-if="appSetting.contentComponent === 'readme'" :title="appSetting.title" :reference="appSetting.reference"  @introduce-author="gotoIntroduceAuthor" />
        <Footmark v-else-if="appSetting.contentComponent === 'footmark'" :title="appSetting.title" :account="userInfo.account" :googleMapApiKey="appSetting.googleMapApiKey" @popup-message="popupMessage" />
        <Finance v-else-if="appSetting.contentComponent === 'finance'" :title="appSetting.title" :account="userInfo.account" />
        <Setting v-else-if="appSetting.contentComponent === 'setting'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" @popup-message="popupMessage" />
        <Chat v-else-if="appSetting.contentComponent === 'chat'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" @popup-message="popupMessage" />
        <Imagen v-else-if="appSetting.contentComponent === 'imagen'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" @popup-message="popupMessage" />
        <PlanTrip v-else-if="appSetting.contentComponent === 'plan_trip'" :title="appSetting.title" :account="userInfo.account" :googleMapApiKey="appSetting.googleMapApiKey" @popup-message="popupMessage" />
        <TripSchedule v-else-if="appSetting.contentComponent === 'trip_schedule'" :title="appSetting.title" :account="userInfo.account" :googleMapApiKey="appSetting.googleMapApiKey" @popup-message="popupMessage" />
        <Author v-else-if="appSetting.contentComponent === 'author'" :title="appSetting.title" />
        <Recollection v-else-if="appSetting.contentComponent === 'recollection'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" @popup-message="popupMessage" />
        <Achievement v-else-if="appSetting.contentComponent === 'achievement'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" />

        <QA v-else-if="appSetting.contentComponent === 'qa'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" @popup-message="popupMessage" />
        <Survey v-else-if="appSetting.contentComponent === 'survey'" :title="appSetting.title" :account="userInfo.account" />
        <Finance_KF v-else-if="appSetting.contentComponent === 'finance_kf'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" @popup-message="popupMessage" />
        <Rule_KF v-else-if="appSetting.contentComponent === 'rule_kf'" :title="appSetting.title" :account="userInfo.account" :funSetting="kf_funSetting" />
        <Activity_KF v-else-if="appSetting.contentComponent === 'activity_kf'" :title="appSetting.title" :account="userInfo.account" :user_role="userInfo.role" :annouce_period_start="kf_funSetting.activity_annouce_period.start" :annouce_period_end="kf_funSetting.activity_annouce_period.end" />
    </div>

    <!-- signin modal -->
    <dialog id="signinModal" class="modal">
        <div class="modal-box h-4/5 max-w-1/1 flex flex-col place-content-center bg-transparent overflow-hidden">

            <div class="h-auto w-10/10 flex flex-col place-content-center rounded-2xl bg-white p-3 z-51">
                <h3 class="text-lg font-bold text-center">Hello!</h3>
                <h4 v-if="signinError" class="text-red-900 text-center mb-5">Error! 登入失敗~ 請檢查登入帳號!</h4>
                <div v-if="!isAutoSignin" class="flex justify-center items-center join">
                    <input id="signinName" type="text" placeholder="What's your name?" class="input input-ghost join-item" :value="tempAccount" @change="keyinAccount" @keyup.enter="signin" autofocus />
                    <button class="btn btn-primary join-item ml-1" @click="signin" >
                        <svg class="w-6 h-6 text-white" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 12H4m12 0-4 4m4-4-4-4m3-4h2a3 3 0 0 1 3 3v10a3 3 0 0 1-3 3h-2"/>
                        </svg>
                    </button>
                </div>
                <div v-if="isAutoSignin" class="flex justify-center items-center">
                    歡迎光臨, 請稍等 <span class="loading loading-dots loading-xs"></span>
                </div>
            </div>
            
            <div id="boxContainer" class="fixed top-0 left-0 w-10/10 h-10/10 flex flex-wrap justify-center items-center">
            </div>
        </div>
    </dialog>
    <!-- userInfo modal -->
    <dialog id="userInfoModal" class="modal">
        <div class="modal-box bg-gray-800/80 rounded-box p-2 w-80 md:w-120">
            <li class="text-white text-lg">{{ userInfo.cname }}</li>
            <li v-if="userInfo.mail" class="text-white text-lg">{{ userInfo.mail }}</li>
            <div class="divider divider-primary"></div>
            <li class="flex flex-row gap-2 w-10/10">
                <button class="btn w-10/10" @click="signout">
                    <span v-if="userInfo.account">
                        {{ userInfo.languages["signout"] }}
                    </span>
                    <svg v-if="userInfo.account" class="w-6 h-6 text-black" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H8m12 0-4 4m4-4-4-4M9 4H7a3 3 0 0 0-3 3v10a3 3 0 0 0 3 3h2"/>
                    </svg>

                    <svg v-if="!userInfo.account" class="w-6 h-6 text-black" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 12H4m12 0-4 4m4-4-4-4m3-4h2a3 3 0 0 1 3 3v10a3 3 0 0 1-3 3h-2"/>
                    </svg>
                    <span v-if="!userInfo.account">
                        {{ userInfo.languages["signin"] }}
                    </span>
                </button>
            </li>
        </div>
        <form method="dialog" class="modal-backdrop">
            <button>close</button>
        </form>
    </dialog>

    <PopupMessage id="alertMsg" class="hidden z-[999]" :status="opObj.status" :message="opObj.message" />
</template>

<style scoped>

</style>
