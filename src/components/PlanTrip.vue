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
        chat("HI");
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

            chat_room_uuid.value = values[0]["chat_room_uuid"];
            let ai_msg = values[0]["message"];
            let speaker = "";
            let short_name = "";
            aiRoles.forEach((roleObj, role_i) => {
                if(roleObj["code"] === values[0]["ai_role"]){
                    speaker = roleObj["name"];
                    short_name = roleObj["short_name"];

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
            }, 100);
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
        chat("幫我列出上次聊的行程");
    }        
    // 開啟 sumup modal
    function openSumupModal(){
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

            let tmpTextAry = values[0]["message"].split("\n");
            if(tmpTextAry.length > 0) tmpTextAry.splice(0, 1);
            if(tmpTextAry.length > 0) tmpTextAry.splice(tmpTextAry.length - 1, 1);
            //console.log("tmpTextAry=", tmpTextAry.join("").trim());
            trip_schedule_obj = JSON.parse(tmpTextAry.join("").trim());
            //console.log("trip_schedule_obj=", trip_schedule_obj);

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


        let geoPromiseAry = [];
        sel_day_sequence_obj.trip_detail_of_day.forEach((tddObj, tdd_i) => {
            geoPromiseAry.push(fetchData({
                api: "get_geocoded_from_google_map",
                data: {
                    location_name: trip_schedule_obj.destination + ( tddObj["subway_station"] ? tddObj["subway_station"] : tddObj["location"] ),
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

            mapCenter_lat.value = googleMapMarks[0].marker.position.lat;
            mapCenter_lng.value = googleMapMarks[0].marker.position.lng;
        });
    }
    // 選擇 TripSumupObj
    function clickSelTripSumup(tdObj){
        // console.log("clickSelTripSumup.tdObj=", tdObj);

        sel_day_sequence.value = tdObj.day_sequence;
        drawGoogleMapMarker();
    }
    // 開啟 InfoWindow
    function openInfoWindow(markObj){
        console.log("openInfoWindow.markObj=", markObj);
        infoWindowObj.isOpen = true;
        infoWindowObj.options = {
            position: markObj.marker.position,
        };
        infoWindowObj.location_name = markObj.location_name;
        infoWindowObj.mark_date = markObj.mark_date;
        infoWindowObj.type = markObj.type;
        infoWindowObj.memo = markObj.memo;
    }
    // 儲存行程規劃
    function saveTripSchedule(){
        console.log("saveTripSchedule.trip_schedule_obj=", trip_schedule_obj);

        let saveTripSchedulePromise = fetchData({
            api: "new_trip_schedule",
            data: {
                account: props.account,
                destination: trip_schedule_obj.destination,
                trip_start_date: trip_schedule_obj.trip_start_date,
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
    function replanTrip(){
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
            chat("HI");
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

<div class="join join-horizontal absolute bottom-5 left-0 w-10/10 justify-start md:justify-center bg-gray-200 px-2 gap-2">
    <button class="btn join-item bg-red-300 text-gray-900 hover:underline hover:bg-gray-900 hover:text-gray-100 btn-circle" title="新話題" @click="openReplanConfirmModal">
        <svg class="size-5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
          <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m15 9-6 6m0-6 6 6m6-3a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
        </svg>
    </button>
    <input type="text" placeholder="想說點什麼呢?" class="input input-info join-item w-6/10" v-model="userMessage" @keyup.enter="send" />
    <button class="btn join-item bg-gray-300 btn-circle hover:bg-blue-300" title="傳送" @click="send">
        <svg class="size-5 text-gray-700 rotate-90" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
            <path fill-rule="evenodd" d="M12 2a1 1 0 0 1 .932.638l7 18a1 1 0 0 1-1.326 1.281L13 19.517V13a1 1 0 1 0-2 0v6.517l-5.606 2.402a1 1 0 0 1-1.326-1.281l7-18A1 1 0 0 1 12 2Z" clip-rule="evenodd"/>
        </svg>
    </button>
    <button class="btn join-item bg-gray-300 btn-circle hover:bg-blue-300" title="列出上次聊的行程" @click="remindPlan">
        <svg class="size-5 text-gray-700" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 15v2a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3v-2M12 4v12m0-12 4 4m-4-4L8 8"/>
        </svg>
    </button>
    <button class="btn join-item bg-gray-300 btn-circle hover:bg-blue-300" title="總結行程" @click="openSumupModal">
        <svg class="size-5 text-gray-700" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-width="2" d="M4.37 7.657c2.063.528 2.396 2.806 3.202 3.87 1.07 1.413 2.075 1.228 3.192 2.644 1.805 2.289 1.312 5.705 1.312 6.705M20 15h-1a4 4 0 0 0-4 4v1M8.587 3.992c0 .822.112 1.886 1.515 2.58 1.402.693 2.918.351 2.918 2.334 0 .276 0 2.008 1.972 2.008 2.026.031 2.026-1.678 2.026-2.008 0-.65.527-.9 1.177-.9H20M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
        </svg>
    </button>
</div>

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
                
                <InfoWindow class="flex flex-col gap-2 pr-5" :options="infoWindowObj.options" v-model:opened="infoWindowObj.isOpen">
                    <h3 class="text-lg text-black">{{ infoWindowObj.mark_date }}</h3>    
                    <h1 class="text-2xl text-black">{{ infoWindowObj.location_name }}</h1>    
                    <h3 class="text-base text-black">{{ infoWindowObj.memo }}</h3>    
                </InfoWindow>

                <AdvancedMarker v-for="(markObj, m_i) in googleMapMarks" :options="markObj.marker" :pin-options="{ background: 'red' }" @click="openInfoWindow(markObj)">
                </AdvancedMarker>        
            </GoogleMap>
        </div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 text-gray-900 hover:underline" @click="closeSumupModal">
                再聊聊
            </button>

            <button class="btn btn-ghost w-1/2 text-gray-900 hover:underline" @click="saveTripSchedule">
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
            <button class="btn btn-ghost w-1/2 text-gray-100 hover:bg-gray-100 hover:text-gray-900 hover:underline" @click="closeReplanConfirmModal">
                話題繼續
            </button>

            <button class="btn btn-ghost w-1/2 text-gray-100 hover:bg-gray-100 hover:text-gray-900 hover:underline" @click="replanTrip">
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
