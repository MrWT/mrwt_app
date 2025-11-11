<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"
    import { GoogleMap, AdvancedMarker, CustomControl, InfoWindow } from 'vue3-google-map'

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        googleMapApiKey: String
    })

    onMounted(() => {
        console.log("PlanTrip mounted.");
        init();
    });

    let appState = ref("");
    let chatModalStatus = ref("CLOSE");
    let chatState = ref("TALKING");
    let userMessage = ref("");    
    // 聊天室 UUID
    let chat_room_uuid = ref("INIT");
    let messages = reactive([]);
    let userInfo = reactive({});
    let aiRoles = reactive([]);
    let currentAiRole = reactive({});

    let trip_schedule_obj = {};
    let tripSumupList = reactive([]);
    let sel_day_sequence = ref("");
    let scheduleList = reactive([]);
    // InfoWindow options
    let infoWindowObj = reactive({});
    // 圖標類型
    let googleMapMarkerType = reactive([]);
    // google map 初始中心點 - 台灣地理中心碑
    let mapCenter_lat = ref(23.974174340321614);
    let mapCenter_lng = ref(120.97984968163026);
    // google map mark
    let googleMapMarks = reactive([]);

    // 初始化 component
    function init(){
        console.log("PlanTrip.init");
        console.log("PlanTrip.props.title", props.title);
        console.log("PlanTrip.props.account", props.account);
        console.log("PlanTrip.props.googleMapApiKey", props.googleMapApiKey);

        fetchInitData();
        //chat("HI");
        remindPlan();
    }
    // 取得初始資料
    function fetchInitData(){
        // 取得 AI 角色
        let fetchAIRolesPromise = fetchData({
            api: "get_ai_role",
        }, "AI");
        // 取得使用者資訊
        let fetchUserInfoPromise = fetchData({
            api: "get_user",
            data: {
                account: props.account,
            }
        });
        // 取得已排定行程清單
        scheduleList.splice(0, scheduleList.length);
        let fetchTripSchedulePromise = fetchData({
            api: "get_trip_schedule",
            data: {
                account: props.account,
            }
        });
        Promise.all([fetchAIRolesPromise, fetchUserInfoPromise, fetchTripSchedulePromise]).then((values) => {
            console.log("fetchInitData.values=", values);
            aiRoles = values[0];
            userInfo = values[1];

            // 建立已排定的行程清單
            {
                values[2].sort((x, y) => {
                    if(x["trip_start_date"] > y["trip_start_date"]){
                        return 1;
                    }
                    if(x["trip_start_date"] < y["trip_start_date"]){
                        return -1;
                    }

                    if(x["destination"] < y["destination"]){
                        return 1;
                    }
                    if(x["destination"] > y["destination"]){
                        return -1;
                    }

                    return 0;
                });
                values[2].forEach((schObj, sch_i) => {
                    let schedule = JSON.parse(schObj["schedule"]);
                    scheduleList.push( schedule );
                });
            }
        });
    }
    // chat with ai
    function chat(message){
        console.log("chat.message=" + message);
        // 關閉全部 modal
        closeAllModal();

        chatState.value = "TALKING";
        let chatPromise = fetchData({
            api: "plan_trip",
            data: {
                account: props.account,
                chat_room_uuid: chat_room_uuid.value,
                message: message,
                time: moment().format("YYYY-MM-DD HH:mm:ss"),
            }
        }, "AI");
        Promise.all([chatPromise]).then((values) => {
            console.log("chatPromise.values=", values);

            try{
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

                setTimeout(() => {
                    let chatBoxElement = document.getElementById("chatBox");
                    chatBoxElement.scrollTo(0, chatBoxElement.scrollHeight);

                    chatState.value = "DONE";
                }, 100);
            }catch(ex){
                let opObj = {
                    message: "剛剛 AI 可能暫離開位置, 麻煩再操作一次.",
                    status: false,
                };
                // 將 message 傳給 App.vue 
                emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data

                chatState.value = "DONE";
            }
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

        chat(userMessage.value);

        setTimeout(() => {
            userMessage.value = "";
            let chatBoxElement = document.getElementById("chatBox");
            chatBoxElement.scrollTo(0, chatBoxElement.scrollHeight);
        }, 100);
    }        
    // 一鍵列出之前聊天內容
    function remindPlan(){
        let chat_msg = "";

        chat_msg += "如果之前的聊天內容中有任何行程相關的話題, ";
        chat_msg += "列出上次聊天的行程詳細內容";
        chat_msg += "回應上次聊天的行程詳細內容前, 請加上'Hello, 我們上次聊到的行程如下'";
        chat_msg += "回應上次聊天的行程詳細內容後, 請加上'有任何我幫得上忙的地方, 隨時跟我說~~";

        chat_msg += "如果之前的聊天內容中沒有任何行程相關的話題, 就直接和 user 打招呼";

        chat( chat_msg );
    }        
    // 開啟 sumup modal
    function openSumupModal(){
        // 關閉全部 Modal
        closeAllModal();

        // 整理聊天內容成"json"
        let chatPromise = fetchData({
            api: "sumup_trip",
            data: {
                account: props.account,
                chat_room_uuid: chat_room_uuid.value,
                message: "",
                time: moment().format("YYYY-MM-DD HH:mm:ss"),
            }
        }, "AI");
        Promise.all([chatPromise]).then((values) => {
            console.log("chatPromise.values=", values);

            let ai_msg = values[0]["message"];
            ai_msg = ai_msg.replace(/```/g, "").replace(/json/g, "").replace(/\n/g, "").trim();
            console.log("ai_msg=", ai_msg);
            trip_schedule_obj = JSON.parse(ai_msg);
            console.log("trip_schedule_obj=", trip_schedule_obj);

            tripSumupList.splice(0, tripSumupList.length);
            trip_schedule_obj.trip_detail.forEach((tdObj, td_i) => {
                tripSumupList.push(tdObj);
            });
            sel_day_sequence.value = tripSumupList[0].day_sequence;

            document.getElementById("sumupModal").showModal();

            drawGoogleMapMarker();
        });
    }
    // 繪畫圖標
    function drawGoogleMapMarker(){
        googleMapMarks.splice(0, googleMapMarks.length);

        let sel_day_sequence_obj = {};
        tripSumupList.forEach((tdObj, td_i) => {
            let day_sequence = tdObj.day_sequence;
            if(day_sequence === sel_day_sequence.value){
                sel_day_sequence_obj = tdObj;
            }
        });

        // 標註每個停靠點
        {
            let geoPromiseAry = [];
            sel_day_sequence_obj.trip_detail_of_day.forEach((tddObj, tdd_i) => {
                geoPromiseAry.push(fetchData({
                    api: "get_geocoded_from_google_map",
                    data: {
                        location_name: trip_schedule_obj.nation + ( tddObj["subway_station"] ? tddObj["subway_station"] : tddObj["location"] ),
                    }
                }));
            });
            Promise.all(geoPromiseAry).then((values) => {
                console.log("geocodedPromise.values=", values);

                values.forEach((geoObj, geo_i) => {
                    let latitude_of_tddObj = geoObj[0];
                    let longitude_of_tddObj = geoObj[1];

                    googleMapMarks.push({
                        location_name: sel_day_sequence_obj.trip_detail_of_day[geo_i]["location"],
                        mark_date: sel_day_sequence_obj.day_sequence + " - " + ((geo_i + 1) < 10 ? "0" : "") + (geo_i + 1),
                        type: "plan_a_trip",
                        memo: sel_day_sequence_obj.trip_detail_of_day[geo_i]["memo"],
                        marker: {
                            position: { 
                                lat: latitude_of_tddObj, 
                                lng: longitude_of_tddObj 
                            },
                            title: sel_day_sequence_obj.trip_detail_of_day[geo_i]["location"],
                        }
                    });
                });
            });
        }

        // 將 google map 中心點設定在目的地
        {
            let fetchCenterPromise = fetchData({
                api: "get_geocoded_from_google_map",
                data: {
                    location_name: trip_schedule_obj.nation + trip_schedule_obj.destination,
                }
            });
            Promise.all([fetchCenterPromise]).then((values) => {
                let latitude_of_tddObj = values[0][0];
                let longitude_of_tddObj = values[0][1];

                // 以目的地為中心點
                mapCenter_lat.value = latitude_of_tddObj;
                mapCenter_lng.value = longitude_of_tddObj;
            });
        }
        
    }
    // 選擇 TripSumupObj
    function clickSelTripSumup(tdObj){
        // console.log("clickSelTripSumup.tdObj=", tdObj);

        sel_day_sequence.value = tdObj.day_sequence;
        drawGoogleMapMarker();
    }
    // 開啟 InfoWindow
    function openInfoWindow(markObj){
        //console.log("openInfoWindow.markObj=", markObj);
        infoWindowObj.isOpen = true;
        infoWindowObj.options = {
            position: markObj.marker.position,
        };
        infoWindowObj.location_name = markObj.location_name;
        infoWindowObj.mark_date = markObj.mark_date;
        infoWindowObj.type = markObj.type;
        infoWindowObj.memo = markObj.memo;
    }
    // 關閉 InfoWindow
    function closeInfoWindow(){
        //console.log("infoWindow.closeclick");
        infoWindowObj.isOpen = false;
    }
    // 儲存行程規劃
    function saveTripSchedule(){
        console.log("saveTripSchedule.trip_schedule_obj=", trip_schedule_obj);

        let saveTripSchedulePromise = fetchData({
            api: "new_trip_schedule",
            data: {
                account: props.account,
                destination: trip_schedule_obj.destination,
                trip_start_date: trip_schedule_obj.trip_start_date ? trip_schedule_obj.trip_start_date : "",
                schedule: JSON.stringify( trip_schedule_obj ),
            }
        });
        Promise.all([saveTripSchedulePromise]).then((values) => {
            console.log("saveTripSchedulePromise.values=", values);

            closeSumupModal();

            let opObj = {
                message: "",
                status: true,
            };
            opObj.status = values[0]["result"];
            if(values[0]["result"] === true){
                opObj.message = "儲存成功";
            }else{
                opObj.message = "Error: " + values[0]["message"];
            }

            // 將 message 傳給 App.vue 
            emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data
        });
    }
    // 開啟 Replan 再確認 modal
    function openReplanConfirmModal(){
        document.getElementById("replanConfirmModal").showModal();
    }
    // 關閉 Replan 再確認 modal
    function closeReplanConfirmModal(){
        document.getElementById("replanConfirmModal").close();
    }
    // 重新規畫 trip
    function replanTrip(json_onScheduleTrip = null){
        console.log("replanTrip");

        let removeTripPromise = fetchData({
            api: "remove_trip",
            data: {
                account: props.account,
            },
        }, "AI");
        Promise.all([removeTripPromise]).then((values) => {
            console.log("removeTripPromise.values=", values);
            // 清空聊天室內容
            messages.splice(0, messages.length);
            // 重新 chat
            chat_room_uuid.value = "INIT";

            if(typeof json_onScheduleTrip === "string"){
                let chat_msg = "";
                chat_msg += "幫我閱讀下列以 json 格式編寫的資料, 並以口語方式將內容回應給使用者,";
                chat_msg += "回應內容前請加上 'Hello, 關於這段旅程的規劃如下:'";
                chat_msg += "回應內容後請加上 '以上, 有需協助的地方, 儘管提出...'";
                chat_msg += json_onScheduleTrip;
                chat(chat_msg);
            }else{
                chat("HI");
            }
            // 關閉 replan 再確認 modal
            closeReplanConfirmModal();
            // 關閉 sumup modal
            closeSumupModal();
        });
    }
    // 關閉 sumup modal
    function closeSumupModal(){
        document.getElementById("sumupModal").close();
    }
    // 修改已排定行程
    function editOnScheduleTrip(schObj){
        console.log("editOnScheduleTrip.schObj=", schObj);
        let json_onScheduleTrip = JSON.stringify(schObj);
        // 清空先前聊天內容, 並列出已排定的旅行
        replanTrip(json_onScheduleTrip);
     
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
        closeReplanConfirmModal();
        closeSumupModal();
    }

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

<!-- 開啟對話 modal -->
<button class="btn btn-circle absolute right-2 bottom-2 z-10 bg-gray-900 text-gray-100 border-0 border-black hover:border-2"  
       :class="{'hidden': chatModalStatus === 'OPEN'}" title="開啟對話 bar" @click="openChatModal">
    <!-- open -->
    <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17h6l3 3v-3h2V9h-2M4 4h11v8H9l-3 3v-3H4V4Z"/>
    </svg>
</button>

<!-- chat modal -->
<dialog id="chatModal" class="modal modal-end md:modal-middle">
    <div class="modal-box h-10/10 w-1/1 md:w-8/10 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span class="text-xl text-gray-900 text-center"></span>
        </div>
        <div class="h-8/10 w-10/10 flex flex-col overflow-y-auto gap-2">
            <div class="w-1/1 flex flex-col gap-2">
                <!-- 開啟新話題 -->
                <button class="btn bg-red-300 text-gray-900 hover:bg-gray-900 hover:text-gray-100 w-1/1" @click="openReplanConfirmModal">
                    <svg class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m15 9-6 6m0-6 6 6m6-3a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
                    </svg>
                    新話題
                </button>
                <!-- 列出之前聊天內容 -->
                <button class="btn bg-gray-300 w-1/1 hover:bg-blue-300" @click="remindPlan">
                    <svg class="size-4 text-gray-700" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 17h6l3 3v-3h2V9h-2M4 4h11v8H9l-3 3v-3H4V4Z"/>
                    </svg>
                    列出之前聊天內容
                </button>
                <!-- 總結對話 -->
                <button class="btn bg-gray-300 w-1/1 hover:bg-blue-300" @click="openSumupModal">
                    <svg class="size-4 text-gray-700" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 5V4a1 1 0 0 0-1-1H8.914a1 1 0 0 0-.707.293L4.293 7.207A1 1 0 0 0 4 7.914V20a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-5M9 3v4a1 1 0 0 1-1 1H4m11.383.772 2.745 2.746m1.215-3.906a2.089 2.089 0 0 1 0 2.953l-6.65 6.646L9 17.95l.739-3.692 6.646-6.646a2.087 2.087 0 0 1 2.958 0Z"/>
                    </svg>
                    總結對話
                </button>
                <!-- 調整已排定的旅行 -->
                <div v-if="scheduleList.length > 0" class="dropdown dropdown-bottom dropdown-end">
                    <div tabindex="0" role="button">
                        <button class="btn bg-gray-300 w-1/1 hover:bg-blue-300">
                            <svg class="size-5 text-gray-700" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 9h6m-6 3h6m-6 3h6M6.996 9h.01m-.01 3h.01m-.01 3h.01M4 5h16a1 1 0 0 1 1 1v12a1 1 0 0 1-1 1H4a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1Z"/>
                            </svg>
                            調整已排定的旅行
                        </button>
                    </div>
                    <ul tabindex="-1" class="dropdown-content menu bg-base-100 border border-gray-900 rounded-box z-1 w-52 p-2 gap-2 shadow-sm">
                        <li class="bg-yellow-100 text-gray-900 p-1 flex flex-col rounded-xl">
                            <div class="text-base md:text-lg">注意!</div>
                            <div class="text-base md:text-lg">調整'已排定的旅行'會清空現有聊天內容!</div>
                        </li>
                        <li v-for="(schObj, sch_i) in scheduleList" class="bg-gray-200">
                            <a @click="editOnScheduleTrip(schObj)" class="flex flex-col">
                                <div>{{ schObj.destination }}</div>
                                <div class="text-xs">( {{ schObj.trip_start_date }} - {{ schObj.trip_end_date }} )</div>
                            </a>
                        </li>
                    </ul>
                </div>
            </div>

            <div class="divider divider-primary"></div>
            <textarea class="textarea w-1/1 h-1/3" placeholder="想說點什麼呢?" v-model="userMessage"></textarea>

            <!-- 傳送訊息 -->
            <button class="btn bg-gray-300 hover:bg-blue-300 w-1/1" @click="send">
                <svg class="size-4 text-gray-700 rotate-90" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                    <path fill-rule="evenodd" d="M12 2a1 1 0 0 1 .932.638l7 18a1 1 0 0 1-1.326 1.281L13 19.517V13a1 1 0 1 0-2 0v6.517l-5.606 2.402a1 1 0 0 1-1.326-1.281l7-18A1 1 0 0 1 12 2Z" clip-rule="evenodd"/>
                </svg>
                傳送訊息
            </button>
        </div>
        <div class="divider divider-primary"></div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/1 bg-gray-900 text-gray-100 hover:bg-gray-100 hover:text-gray-900" @click="closeChatModal">
                關閉
            </button>
        </div>
    </div>
</dialog>

<!-- sumup modal -->
<dialog id="sumupModal" class="modal">
    <div class="modal-box h-10/10 w-10/10 max-w-3xl flex flex-col bg-neutral-100">
        <div class="h-1/10 w-10/10 flex flex-row overflow-x-auto">
            <button v-for="(tdObj, td_i) in tripSumupList" class="btn btn-ghost" :class="{ 'border-black': sel_day_sequence === tdObj.day_sequence}" @click="clickSelTripSumup(tdObj)">
                {{ tdObj.day_sequence }}
            </button>
        </div>
        <div class="h-4/10 w-10/10 flex flex-col overflow-y-auto">
            <div v-for="(tdObj, td_i) in tripSumupList">
                <ul class="list bg-yellow-100 rounded-box shadow-md">
                    <li v-if="sel_day_sequence === tdObj.day_sequence" v-for="(tddObj, tdd_i) in tdObj.trip_detail_of_day" class="list-row">
                        <div class="text-4xl font-thin opacity-30 tabular-nums">{{ ((tdd_i + 1) < 10 ? "0" : "") + (tdd_i + 1) }}</div>
                        <div class="list-col-grow">
                            <div class="text-lg">
                                {{ tddObj.location }}
                                <span v-if="tddObj.subway_station">{{ " - 地鐵: " + tddObj.subway_station }}</span>
                            </div>
                            <div class="text-md uppercase font-semibold">{{ tddObj.memo }}</div>
                        </div>
                    </li>
                </ul>

            </div>
        </div>
        <div class="h-5/10 w-10/10 flex flex-col mt-1">
            <GoogleMap class="w-10/10 h-10/10 border"
                mapId="PLAN_TRIP_MAP_ID"
                :api-key="props.googleMapApiKey"
                :center="{ lat: mapCenter_lat, lng: mapCenter_lng }"
                :zoom="10"
                :mapTypeControl = "false" 
                :streetViewControl = "false"
                >
                
                <InfoWindow class="flex flex-col gap-2 pr-5 w-1/1" :options="infoWindowObj.options" v-model="infoWindowObj.isOpen" @closeclick="closeInfoWindow">
                    <div class="text-lg text-black">{{ infoWindowObj.mark_date }} - {{ infoWindowObj.location_name }}</div>    
                    <div class="text-base text-black">{{ infoWindowObj.memo }}</div>    
                </InfoWindow>

                <AdvancedMarker v-for="(markObj, m_i) in googleMapMarks" :options="markObj.marker" :pin-options="{ background: 'red' }" @click="openInfoWindow(markObj)">
                </AdvancedMarker>        
            </GoogleMap>
        </div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 bg-gray-900 text-gray-100 hover:underline hover:bg-gray-100 hover:text-black" @click="closeSumupModal">
                <svg class="size-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 9h5m3 0h2M7 12h2m3 0h5M5 5h14a1 1 0 0 1 1 1v9a1 1 0 0 1-1 1h-6.616a1 1 0 0 0-.67.257l-2.88 2.592A.5.5 0 0 1 8 18.477V17a1 1 0 0 0-1-1H5a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1Z"/>
                </svg>
                再聊聊
            </button>

            <button class="btn btn-ghost w-1/2 bg-gray-900 text-gray-100 hover:underline hover:bg-gray-100 hover:text-black" @click="saveTripSchedule">
                <svg class="size-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linejoin="round" stroke-width="2" d="M10 3v4a1 1 0 0 1-1 1H5m14-4v16a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V7.914a1 1 0 0 1 .293-.707l3.914-3.914A1 1 0 0 1 9.914 3H18a1 1 0 0 1 1 1Z"/>
                </svg>
                儲存規畫
            </button>
        </div>
    </div>
    <form method="dialog" class="modal-backdrop">
        <button>close</button>
    </form>
</dialog>

<!-- replanConfirm modal -->
<dialog id="replanConfirmModal" class="modal">
    <div class="modal-box h-3/10 w-10/10 flex flex-col bg-neutral-500">
        <div class="h-10/10 w-10/10 text-center text-black font-black">
            <span class="text-2xl">再跟您確認一次~</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 text-gray-900 bg-gray-100 hover:bg-gray-500 hover:text-gray-100 hover:underline" @click="closeReplanConfirmModal">
                話題繼續
            </button>

            <button class="btn btn-ghost w-1/2 text-gray-900 bg-gray-100 hover:bg-gray-500 hover:text-gray-100 hover:underline" @click="replanTrip">
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
