<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    import { gsap } from "gsap"
    import { DrawSVGPlugin } from "gsap/DrawSVGPlugin"
    import { MotionPathPlugin } from 'gsap/MotionPathPlugin'
    import { getRandomNumber } from "@/composables/random"

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        cname: String,
        user_role: String,
        focus_news_topic: String,
    })

    onMounted(() => {
        console.log("gallery.mounted");
        init();

        setTimeout(() => {
            window.dispatchEvent(new Event('resize'));
        }, 1000);

    });

    let appState = ref("");
    let yesterday = ref("");
    let topicList = reactive([]);
    let topicBgColorCount = ref(10);
    let newsList = reactive({});

    let countryOptions = reactive([
        "台灣",
        "韓國",
        "日本",
        "中國",
        "美國",
        "國際",
    ]);
    let dataSourceOptions = reactive([
        "康健雜誌", 
        "heho健康", 
        "經濟日報",
        "自由時報", 
        "聯合報", 
        "三立新聞網", 
        "TVBS新聞網", 
        "CARNEWS車訊網", 
        "U-CAR", 
        "國王車訊", 
        "小老婆汽機車資訊網",
        "交通部中央氣象署",
    ]);
    let selTopicObj = reactive({
        id: "",
        key: "",
        desc: "",
        seq: "",
        prompt: "",
        data_source: "",
        dataSourceList: [],
        max_news_count: "",
    });
    let selTopicIndex = ref(-1);

    let setSubscribe = ref(false);

    let reDlTopicKey = ref("");
    let setTopicDesc = ref("");
    let setTopicObj = reactive({
        id: "",
        key: "",
        desc: "",
        kind: "",
        seq: "",
        prompt: "",
        data_source: "",
        dataSourceList: [],
        country: "台灣",
        max_news_count: "",
    });

    // 初始化 component
    function init(){
        console.log("gallery.init");
        console.log("gallery.props.title=", props.title);
        console.log("gallery.props.account=", props.account);
        console.log("gallery.props.cname=", props.cname);
        console.log("gallery.props.user_role=", props.user_role);
        console.log("gallery.props.focus_news_topic=", props.focus_news_topic);
        if(props.account && props.user_role){
            appState.value = props.account === "KUOFAMILY" || props.user_role === "admin_kf" ? "k_funds" : "normal";
        }

        yesterday.value = moment().add(-1, "d").format("YYYY-MM-DD");

        if(appState.value === "k_funds"){
            initGsap();
        }else{
            fetchInitData();
        }
    }    
    // 初始化 - 動畫效果
    function initGsap(){
        gsap.registerPlugin(DrawSVGPlugin);
        gsap.registerPlugin(MotionPathPlugin);

        // 候選顏色
        let candidateColors = ["green", "blue", "purple", "gold", "peru", "blanchedalmond", "blueviolet", "goldenrod"];
        let durations = [];
        for(let d_i = 0; d_i < 10; d_i++){
            durations.push( getRandomNumber(10, 20)/10 );
        }

        setTimeout(() => {
            let tl_back = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });
            let tl_word = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });

            for(let c_i = 0; c_i < candidateColors.length; c_i++){
                // 郭字背景
                tl_back.to("#" + "kuoBack", {
                    id: "tween_" + "kuoBack",
                    duration: getRandomNumber(30, 40)/10,
                    background: candidateColors[c_i],
                    ease: "bounce.inOut",
                });

                // 郭字
                tl_word.to("#" + "kuoWord", {
                    id: "tween_" + "kuoWord",
                    duration: getRandomNumber(20, 30)/10,
                    color: candidateColors[ candidateColors.length - 1 - c_i],
                    //ease: "bounce.inOut",
                });
            }

            // 玩 SVG
            let tl_path = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });
            let tl_circle = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });
            {
                tl_path.from("#myPath", {
                    drawSVG: false, 
                    duration: 2, 
                    repeat: -1,
                    repeatDelay: 3,
                    yoyo: true,
                    ease: "power1.inOut"
                });
                for(let c_i = 1; c_i <= 5; c_i++){
                    tl_circle.to("#myCircle" + c_i, {
                        duration: 2, 
                        repeat: -1,
                        repeatDelay: 3,
                        yoyo: true,
                        ease: "power1.inOut",
                        motionPath:{
                            path: "#myPath",
                            align: "#myPath",
                            autoRotate: true,
                            alignOrigin: [0.5, 0.5]
                        }
                    });
                }
            }
        }, 100);
    }
    // 取得初始資料 - 取得新聞資訊
    function fetchInitData(){
        // 清空資料
        {
            topicList.splice(0, topicList.length);
        }
        // 取得全部主題
        let fetchPromise_allTopic = fetchData({
            api: "get_all_topic",
            data: {
                account: props.account,
            }
        });
        Promise.all([fetchPromise_allTopic]).then((values) => {
            console.log("getUserSubscription.values=", values);

            values[0].forEach((topicObj, topic_i) => {
                topicList.push({
                    id: topicObj["id"],
                    key: topicObj["key"],
                    desc: topicObj["desc"],
                    kind: topicObj["kind"],
                    prompt: topicObj["prompt"],
                    data_source: topicObj["data_source"],
                    max_news_count: topicObj["max_news_count"],
                    seq: topicObj["seq"],
                    sel: props.focus_news_topic && props.focus_news_topic.indexOf(topicObj["key"]) >= 0 ? true : false,
                });

                newsList[topicObj["key"]] = [];
            });

            topicList.sort((x, y) => {
                if(x["seq"] > y["seq"]) return 1;
                if(x["seq"] < y["seq"]) return -1;
                return 0;
            });

            // 回到清單
            reDlTopicKey.value = "";
            backToBlock();
        });
    }
    // 選擇關注的 Topic
    function selectTopic(theTopic, theIndex){
        console.log("selectTopic.theTopic=", theTopic, theIndex);

        if(reDlTopicKey.value === theTopic.key){
            // 正在重新下載中... 不做反應
        }else{
            if(setSubscribe.value){
                // 訂閱/取消訂閱主題
                let new_userFocusTopic = "";
                topicList.forEach((topicObj, topic_i) => {
                    if(topicObj["key"] === theTopic["key"]){
                        topicObj["sel"] = !topicObj["sel"];
                    }

                    if(topicObj["sel"] === true){
                        new_userFocusTopic += (new_userFocusTopic.length > 0 ? "," : "") + topicObj["key"];
                    }
                });

                //console.log("new_userFocusTopic=" + new_userFocusTopic);
                let fetchPromise_setUserTopic = fetchData({
                    api: "set_user_topic",
                    data: {
                        account: props.account,
                        focus_topic: new_userFocusTopic,
                    }
                });
                Promise.all([fetchPromise_setUserTopic]).then((values) => {
                    //console.log("fetchPromise_setUserTopic.values=", values);
                    toggleSubscribe();
                });
            }else{
                // 查看指定主題的新聞清單
                selTopicIndex.value = theIndex;

                selTopicObj["id"] = theTopic["id"];
                selTopicObj["key"] = theTopic["key"];
                selTopicObj["desc"] = theTopic["desc"];
                selTopicObj["seq"] = theTopic["seq"];
                selTopicObj["prompt"] = theTopic["prompt"];
                selTopicObj["data_source"] = theTopic["data_source"];
                selTopicObj["dataSourceList"] = theTopic["data_source"].split(",");
                selTopicObj["max_news_count"] = theTopic["max_news_count"];

                setTopicDesc.value = theTopic["desc"];
                setTopicObj["id"] = theTopic["id"];
                setTopicObj["key"] = theTopic["key"];
                setTopicObj["desc"] = theTopic["desc"];
                setTopicObj["kind"] = theTopic["kind"];
                setTopicObj["seq"] = theTopic["seq"];
                setTopicObj["prompt"] = theTopic["prompt"];
                setTopicObj["data_source"] = theTopic["data_source"];
                setTopicObj["dataSourceList"] = theTopic["data_source"].split(",");
                setTopicObj["max_news_count"] = theTopic["max_news_count"];

                // 取得新聞清單
                // 清空資料
                {
                    Object.keys( newsList ).forEach((key, key_i) => {
                        newsList[key].splice(0, newsList[key].length);
                    });
                }
                let fetchNewsPromise = fetchData({
                    api: "get_news",
                    data: {
                        account: props.account,
                        focus_topic: theTopic["key"],
                    }
                });
                Promise.all([fetchNewsPromise]).then((values) => {
                    console.log("fetchNewsPromise.values=", values);

                    values[0].forEach((newsObj, news_i) => {
                        let contentList = [];
                        try{
                            let content = "";

                            // 去投去尾
                            {
                                let start_index = newsObj["content"].indexOf("```json");
                                start_index = start_index === -1 ? 0 : start_index;
                                let end_index = newsObj["content"].indexOf("```", start_index + 6);
                                end_index = end_index === -1 ? newsObj["content"].length : (end_index + 2);
                                content = newsObj["content"].substr(start_index, end_index - start_index + 1);
                            }
                            content = content.replace(/```json/g, "").replace(/```/g, "");

                            contentList = JSON.parse(content);
                            contentList.forEach((contentObj, content_i) => {
                                if(!contentObj.hasOwnProperty("keyword")){
                                    contentObj["keyword"] = "";
                                }
                            });
                        }catch(ex){
                            console.log("newsObj.key=", newsObj.key);
                            console.log("newsObj.content=", newsObj.content);
                            console.error("JSON.parse(content).ex=", ex);
                        }
                        newsList[newsObj["key"]] = contentList;
                    });
                });
            }
        }
    }
    // 回到清單
    function backToBlock(){
        selTopicIndex.value = -1;

        selTopicObj["key"] = "";
        selTopicObj["desc"] = "";
        selTopicObj["seq"] = "";
    }
    // 請 AI 重新取得指定 topic 的新聞資料
    function reDownloadSpecifyTopic(topic_key){
        console.log("reDownloadSpecifyTopic.topic_key=", topic_key);
        reDlTopicKey.value = topic_key;

        // 請 AI 重新取得指定 topic 的新聞資料
        let reDownloadPromise = fetchData({
            api: "fetch_daily_news",
            data: {
                account: props.account,
                specify_topic: topic_key,
            }
        }, "AI");
        Promise.all([reDownloadPromise]).then((values) => {
            console.log("reDownloadPromise.values=", values);

            fetchInitData();
        });
    }
    // 另外搜尋新聞相關 keyword
    function browseNewsKeyword(selNewsObj){
        console.log("browseNewsKeyword.selNewsObj=", selNewsObj);

        let q = selNewsObj["keyword"].trim().replace(/,/g, "+");
        if(selNewsObj["source"]){
            q += "+" + selNewsObj["source"];
        }
        window.open("https://www.google.com/search?q=" + q, "_blank");
    }
    // 訂閱設定關閉
    function toggleSubscribe(){
        setSubscribe.value = !setSubscribe.value;
    }
    // 開啟 setting modal
    function openModal_setting(){
        console.log("openModal_setting.setTopicObj=", setTopicObj);
        document.getElementById("settingModal").showModal();
    }
    // 關閉 setting modal
    function closeModal_setting(){
        document.getElementById("settingModal").close();
    }
    // 組合給予 AI 的指令
    function combineSetting(){
        setTopicObj.data_source = setTopicObj.dataSourceList.join(",");

        let prompt = "";
        prompt += "'" + setTopicObj.dataSourceList.join(" / ") + "'的"
        prompt += setTopicObj.country;
        prompt += setTopicObj.kind + "資訊";
        prompt += "( 最多" + setTopicObj.max_news_count + "則摘要 )";
        setTopicObj.prompt = prompt;
    }
    // 檢查 setting 內容
    function checkSetting(){
        let exeRst = {
            result: true,
            message: "",
        };

        if(setTopicObj.dataSourceList.length === 0){
            exeRst.result = false;
            exeRst.message = "資料來源, 請至少選擇一項 !";
            return exeRst;
        }
        if(!setTopicObj.desc){
            exeRst.result = false;
            exeRst.message = "請輸入方塊名稱 !";
            return exeRst;
        }
        if(!setTopicObj.kind){
            exeRst.result = false;
            exeRst.message = "請輸入新聞分類 !";
            return exeRst;
        }

        return exeRst;
    }
    // 儲存 setting
    function saveSetting(){
        // 組合給予 AI 的指令
        combineSetting();
        console.log("saveSetting.setTopicObj=", setTopicObj);

        let checkRst = checkSetting();
        if(checkRst.result){
            backToBlock();
            // 更新主題新聞取得資料條件
            let fetchPromise_updPrompt = fetchData({
                api: "update_topic_prompt",
                data: setTopicObj
            });
            Promise.all([fetchPromise_updPrompt]).then((values) => {
                console.log("fetchPromise_updPrompt.values=", values);
                closeModal_setting();
                // 請 AI 重新取得指定 topic 的新聞資料
                reDownloadSpecifyTopic(setTopicObj.key);
            });
        }else{
            // 將 message 傳給 App.vue 
            emit('popupMessage', false, checkRst.message); // Emitting the event with data
        }
    }
</script>

<template>
    <!-- 一般使用者 -->
    <div v-if="appState === 'normal' && topicList.length > 0 && selTopicIndex === -1" 
        class="text-3xl w-1/1 bg-zinc-500/50 rounded-lg mb-2 text-end">
        <button v-if="!setSubscribe" class="btn btn-ghost hover:bg-transparent" @click="toggleSubscribe">
            <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                <path stroke="currentColor" stroke-linecap="round" stroke-width="2" d="M6 4v10m0 0a2 2 0 1 0 0 4m0-4a2 2 0 1 1 0 4m0 0v2m6-16v2m0 0a2 2 0 1 0 0 4m0-4a2 2 0 1 1 0 4m0 0v10m6-16v10m0 0a2 2 0 1 0 0 4m0-4a2 2 0 1 1 0 4m0 0v2"/>
            </svg>
            訂閱調整
        </button>
        <button v-if="setSubscribe" class="btn btn-ghost hover:bg-transparent" @click="toggleSubscribe">
            <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 4h3a1 1 0 0 1 1 1v15a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V5a1 1 0 0 1 1-1h3m0 3h6m-6 7 2 2 4-4m-5-9v4h4V3h-4Z"/>
            </svg>
            完成調整
        </button>
    </div>

    <div v-if="appState === 'normal'" class="w-1/1 h-11/12 overflow-y-auto">
        <!-- 主題方塊 -->
        <div v-if="selTopicIndex === -1" class="grid grid-cols-2 md:grid-cols-4 gap-2">
            <!-- 主題選單 -->
            <div v-for="(topicObj, topic_i) in topicList" 
                class="h-50 cursor-pointer rounded-2xl text-2xl text-center content-center" 
                :class="{
                    'bg-gray-200/70 text-gray-500': setSubscribe && !topicObj.sel || reDlTopicKey === topicObj.key,
                    'hidden': !setSubscribe && !topicObj.sel,
                    'bg-lime-200/80': topicObj.sel && topic_i % topicBgColorCount === 0, 
                    'bg-amber-200/80': topicObj.sel && topic_i % topicBgColorCount === 1, 
                    'bg-indigo-200/80': topicObj.sel && topic_i % topicBgColorCount === 2, 
                    'bg-fuchsia-200/80': topicObj.sel && topic_i % topicBgColorCount === 3, 
                    'bg-red-200/80': topicObj.sel && topic_i % topicBgColorCount === 4, 
                    'bg-orange-200/80': topicObj.sel && topic_i % topicBgColorCount === 5, 
                    'bg-emerald-200/80': topicObj.sel && topic_i % topicBgColorCount === 6, 
                    'bg-teal-200/80': topicObj.sel && topic_i % topicBgColorCount === 7, 
                    'bg-cyan-200/80': topicObj.sel && topic_i % topicBgColorCount === 8, 
                    'bg-rose-200/80': topicObj.sel && topic_i % topicBgColorCount === 9, 
                }"                
                @click="selectTopic(topicObj, topic_i)" >
                {{ topicObj.desc }}
                <span v-if="reDlTopicKey === topicObj.key">
                    <br /><span class="loading loading-infinity loading-xl"></span>
                </span>
                <!-- 設定訂閱時, 才會出現 -->
                <span v-if="setSubscribe && !topicObj.sel" class="text-base">
                    <br />{{ "( 按一下完成訂閱 )" }}
                </span>
                <span v-if="setSubscribe && topicObj.sel" class="text-base">
                    <br />{{ "( 按一下取消訂閱 )" }}
                </span>
            </div>
        </div>
        <!-- 新聞清單 -->
        <ul v-if="selTopicIndex > -1 && !setSubscribe" 
            class="list bg-base-100 rounded-box shadow-2xl">
            <li class="p-4 pb-2 tracking-wide flex flex-row items-end text-gray-900 rounded-xl shadow-2xl z-10" 
                :class="{
                    'bg-lime-200/80': selTopicIndex % topicBgColorCount === 0, 
                    'bg-amber-200/80': selTopicIndex % topicBgColorCount === 1, 
                    'bg-indigo-200/80': selTopicIndex % topicBgColorCount === 2, 
                    'bg-fuchsia-200/80': selTopicIndex % topicBgColorCount === 3, 
                    'bg-red-200/80': selTopicIndex % topicBgColorCount === 4, 
                    'bg-orange-200/80': selTopicIndex % topicBgColorCount === 5, 
                    'bg-emerald-200/80': selTopicIndex % topicBgColorCount === 6, 
                    'bg-teal-200/80': selTopicIndex % topicBgColorCount === 7, 
                    'bg-cyan-200/80': selTopicIndex % topicBgColorCount === 8, 
                    'bg-rose-200/80': selTopicIndex % topicBgColorCount === 9, 
                }"                
                style="position: sticky; top:0;">
                <div class="flex-1 flex flex-row items-center">
                    <div class="font-black text-xl mr-5 flex flex-col">
                        {{ selTopicObj.desc }}
                        <span class="text-sm">
                            ( 共 {{ newsList[selTopicObj.key].length }} 則 )
                        </span>
                    </div> 
                </div>
                <div class="flex-none">
                    <button class="btn btn-ghost hover:bg-transparent" title="主題清單" @click="backToBlock">
                        <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.143 4H4.857A.857.857 0 0 0 4 4.857v4.286c0 .473.384.857.857.857h4.286A.857.857 0 0 0 10 9.143V4.857A.857.857 0 0 0 9.143 4Zm10 0h-4.286a.857.857 0 0 0-.857.857v4.286c0 .473.384.857.857.857h4.286A.857.857 0 0 0 20 9.143V4.857A.857.857 0 0 0 19.143 4Zm-10 10H4.857a.857.857 0 0 0-.857.857v4.286c0 .473.384.857.857.857h4.286a.857.857 0 0 0 .857-.857v-4.286A.857.857 0 0 0 9.143 14Zm10 0h-4.286a.857.857 0 0 0-.857.857v4.286c0 .473.384.857.857.857h4.286a.857.857 0 0 0 .857-.857v-4.286a.857.857 0 0 0-.857-.857Z"/>
                        </svg>
                    </button>
                </div>
            </li>
            <li v-if="newsList[selTopicObj.key].length === 0" class="list-row">
                <div class="list-col-grow">
                    <div class="text-black text-lg text-center">{{ "很抱歉 ! 不明原因, 漏了資料 !" }}</div>
                    <div class="text-black text-lg text-center">{{ "可再手動按最下方的'下載'喔 !" }}</div>
                </div>
            </li>
            <li v-for="(newsObj, news_i) in newsList[selTopicObj.key]" class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">
                    {{ ((news_i+1) < 10 ? "0" : "") + (news_i+1) }}
                </div>
                <div class="list-col-grow">
                    <div class="text-black underline text-lg">
                        <span v-if="newsObj.keyword.length === 0">
                            {{ newsObj["topic"] }}
                        </span>

                        <a v-if="newsObj.keyword.length > 0" class="link link-hover cursor-pointer flex flex-row gap-4 items-center text-blue-900" @click="browseNewsKeyword(newsObj)">
                            {{ newsObj["topic"] }}
    
                            <svg class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                                <path stroke="currentColor" stroke-linecap="round" stroke-width="2" d="m21 21-3.5-3.5M17 10a7 7 0 1 1-14 0 7 7 0 0 1 14 0Z"/>
                            </svg>
                        </a>

                        <div v-if="newsObj['source']" class="text-sm mt-1">
                            {{ newsObj["source"] }}
                        </div>
                    </div>
                    <div class="text-md mt-2">{{ newsObj["content"] }}</div>
                    <div v-if="newsObj['keyword']" class="text-md mt-4">
                        # {{ newsObj["keyword"] }}
                    </div>
                </div>
            </li>
            <li class="list-row">
                <div class="list-col-grow">
                    <div class="bg-gray-200 text-black text-lg flex flex-row items-center">                        
                        <div class="flex-1 text-center ">
                            <div>{{ "點擊'方塊', 返回'主題清單'" }}</div>
                        </div>
                        <div class="flex-none">
                            <button class="btn btn-ghost hover:bg-transparent" title="主題清單" @click="backToBlock">
                                <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.143 4H4.857A.857.857 0 0 0 4 4.857v4.286c0 .473.384.857.857.857h4.286A.857.857 0 0 0 10 9.143V4.857A.857.857 0 0 0 9.143 4Zm10 0h-4.286a.857.857 0 0 0-.857.857v4.286c0 .473.384.857.857.857h4.286A.857.857 0 0 0 20 9.143V4.857A.857.857 0 0 0 19.143 4Zm-10 10H4.857a.857.857 0 0 0-.857.857v4.286c0 .473.384.857.857.857h4.286a.857.857 0 0 0 .857-.857v-4.286A.857.857 0 0 0 9.143 14Zm10 0h-4.286a.857.857 0 0 0-.857.857v4.286c0 .473.384.857.857.857h4.286a.857.857 0 0 0 .857-.857v-4.286a.857.857 0 0 0-.857-.857Z"/>
                                </svg>
                            </button>
                        </div>
                    </div>
                </div>
            </li>
            <li class="list-row">
                <div class="list-col-grow">
                    <div class="bg-gray-200 text-black text-lg flex flex-row items-center">                        
                        <div class="flex-1 text-center ">
                            <div>{{ "點擊'鉛筆', 設定新聞來源條件" }}</div>
                        </div>
                        <div class="flex-none">
                            <button class="btn btn-ghost hover:bg-transparent" title="設定內容" @click="openModal_setting(selTopicObj)">
                                <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.779 17.779 4.36 19.918 6.5 13.5m4.279 4.279 8.364-8.643a3.027 3.027 0 0 0-2.14-5.165 3.03 3.03 0 0 0-2.14.886L6.5 13.5m4.279 4.279L6.499 13.5m2.14 2.14 6.213-6.504M12.75 7.04 17 11.28"/>
                                </svg>
                            </button>
                        </div>
                    </div>
                </div>
            </li>
            <li class="list-row">
                <div class="list-col-grow">
                    <div class="bg-gray-200 text-black text-lg flex flex-row items-center">                        
                        <div class="flex-1 text-center ">
                            <div>{{ "點擊'下載', 重新下載'新聞清單'" }}</div>
                        </div>
                        <div class="flex-none">
                            <button class="btn btn-ghost hover:bg-transparent" title="重新下載" @click="reDownloadSpecifyTopic(selTopicObj.key)">
                                <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 13V4M7 14H5a1 1 0 0 0-1 1v4a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1v-4a1 1 0 0 0-1-1h-2m-1-5-4 5-4-5m9 8h.01"/>
                                </svg>
                            </button>
                        </div>
                    </div>
                </div>
            </li>
        </ul>
    </div>

    <!-- setting modal -->
    <dialog id="settingModal" class="modal modal-top">
        <div class="modal-box h-5/6 w-1/1 md:w-1/2 md:h-1/1 flex flex-col bg-neutral-100">
            <div class="flex flex-col justify-center">
                <span class="text-xl text-gray-900 text-center">{{ setTopicDesc }}</span>
                <div class="divider divider-primary"></div>
            </div>
            <div class="h-1/1 w-1/1 flex flex-col overflow-y-auto gap-4 md:gap-1">
                <div class="w-1/1 h-1/1 flex flex-col gap-3">
                    <div class="w-1/1 flex flex-col">
                        <span class="w-1/1 bg-zinc-200/50 p-2">
                            方塊名稱(呈現在方塊的名稱):
                        </span>
                        <input type="text" class="w-1/1 border" v-model="setTopicObj.desc" @keyup="combineSetting" />
                    </div>
                    <div class="w-1/1 flex flex-col">
                        <span class="w-1/1 bg-rose-200/50 p-2">
                            新聞區域:
                        </span>
                        <div class="w-1/1 grid grid-cols-2 md:grid-cols-3 gap-1">
                            <label v-for="(option, option_i) in countryOptions">
                                <input type="radio" :value="option" v-model="setTopicObj.country" @change="combineSetting" />
                                {{ option }}
                            </label>
                        </div>
                    </div>
                    <div class="w-1/1 flex flex-col">
                        <span class="w-1/1 bg-rose-200/50 p-2">
                            新聞分類:
                        </span>
                        <input type="text" class="w-1/1 border" v-model="setTopicObj.kind" @keyup="combineSetting" />
                    </div>
                    <div class="w-1/1 flex flex-col">
                        <span class="w-1/1 bg-rose-200/50 p-2">
                            資料來源:
                        </span>
                        <div class="w-1/1 grid grid-cols-2 md:grid-cols-3 gap-1">
                            <label v-for="(option, option_i) in dataSourceOptions">
                                <input type="checkbox" :value="option" v-model="setTopicObj.dataSourceList" @change="combineSetting" />
                                {{ option }}
                            </label>
                        </div>
                    </div>
                    <div class="w-1/1 flex flex-col">
                        <span class="w-1/1 bg-rose-200/50 p-2">
                            最多則數:
                        </span>
                        <div class="w-1/1">
                            <input type="range" min="1" max="10" value="10" class="range w-1/1" step="1" v-model="setTopicObj.max_news_count" @change="combineSetting" />
                            <div class="flex justify-between px-2.5 mt-2 text-xs">
                                <span>1</span>
                                <span>2</span>
                                <span>3</span>
                                <span>4</span>
                                <span>5</span>
                                <span>6</span>
                                <span>7</span>
                                <span>8</span>
                                <span>9</span>
                                <span>10</span>
                            </div>
                        </div>
                    </div>
                    <div class="w-1/1 flex flex-col">
                        <span class="w-1/1 bg-rose-200/50 p-2">
                            預覽要給予 AI 的指令:
                        </span>
                        <textarea class="w-1/1 h-1/1 border" disabled>{{ setTopicObj.prompt }}</textarea>
                    </div>
                </div>
            </div>
            <div class="divider divider-primary"></div>
            <div class="modal-action">
                <button class="btn btn-ghost bg-gray-900 text-gray-100 hover:bg-gray-100 hover:text-gray-900 w-1/2" 
                        @click="closeModal_setting">
                    <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m15 9-6 6m0-6 6 6m6-3a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
                    </svg>
                    關閉
                </button>
                <button class="btn btn-ghost bg-gray-300 hover:bg-blue-300 w-1/2"
                        @click="saveSetting">
                    <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 3v4a1 1 0 0 1-1 1H5m4 6 2 2 4-4m4-8v16a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V7.914a1 1 0 0 1 .293-.707l3.914-3.914A1 1 0 0 1 9.914 3H18a1 1 0 0 1 1 1Z"/>
                    </svg>
                    儲存
                </button>
            </div>
        </div>
        <form method="dialog" class="modal-backdrop">
            <button>close</button>
        </form>
    </dialog>

    <!-- 郭家基金 - 使用者 -->
    <div v-if="appState === 'k_funds'" id="kuoBack" class="w-10/10 h-10/10 flex flex-col justify-center items-center bg-gray-950">
        <div id="kuoWord" class="text-9xl text-yellow-500">{{ "郭" }}</div>

        <svg width="300" height="100" viewBox="0 0 300 100">
            <path id="myPath" d="M0 0 Q150 100,300 0" fill="none" stroke="#88ce02" stroke-width="3px" />
            <circle id="myCircle1" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle2" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle3" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle4" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle5" cx="0" cy="0" r="10" fill="#88ce02" />
        </svg>
    </div>
    
</template>

<style scoped>

</style>
