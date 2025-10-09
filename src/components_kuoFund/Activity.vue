<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String,
        googleMapApiKey: String, 
    })

    onMounted(() => {
        console.log("Activity_KF mounted.");
        init();
    });

    let appState = ref("");
    let activity = reactive({});
    // google map 初始中心點 - 台灣地理中心碑
    let googleMapCenter = reactive({
        lat: 23.974174340321614, lng: 120.97984968163026
    });
    let manualPages = reactive([]);

    // 初始化 component
    function init(){
        console.log("Activity_KF.init");
        console.log("Activity_KF.props.title=", props.title);
        console.log("Activity_KF.props.account=", props.account);
        console.log("Activity_KF.props.googleMapApiKey=", props.googleMapApiKey);

        get_activity_kuoFunds();
    }   
    // 取得活動清單
    function get_activity_kuoFunds(){
         let getActivityKFPromise = fetchData({
            api: "get_activiy_kf",
        });
        Promise.all([getActivityKFPromise]).then((values) => {
            //console.log("getActivityKFPromise.values=", values);
            activity = values[0][0];
            console.log("get_activity_kuoFunds.activity=", activity);

            googleMapCenter.lat = activity["latitude"];
            googleMapCenter.lng = activity["longitude"];

            manualPages.splice(0, manualPages.length);
            activity.pages.forEach((pageObj, page_i) => {
                manualPages.push({
                    src: pageObj["page"],
                    alt: pageObj["memo"],
                });
            });
        });
    }
    // 開啟 google map 方便設定導航
    function openGoogleMap(){
        const mapsUrl = `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent( activity.location_name )}`;
        window.open(mapsUrl, '_blank');
    }
    
</script>

<template>

<div class="w-10/10 h-10/10 flex flex-col gap-2">
    <div class="w-10/10 h-3/10 p-1">
        <div class="divider">活動資訊</div>
        <div class="w-10/10 bg-emerald-200 text-gray-900 text-2xl text-center font-black">{{ activity.location_name }}</div>
        <div class="w-10/10 mt-2 text-gray-900 text-xl flex justify-center">
            <svg class="size-6 text-gray-800 mr-2" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
            </svg>

            {{ activity.date }}
        </div>
        <div class="w-10/10 mt-2 text-gray-900 text-lg flex justify-center cursor-pointer" @click="openGoogleMap">
            {{ activity.address }}

            <svg class="size-6 text-red-800 ml-2" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 13a3 3 0 1 0 0-6 3 3 0 0 0 0 6Z"/>
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.8 13.938h-.011a7 7 0 1 0-11.464.144h-.016l.14.171c.1.127.2.251.3.371L12 21l5.13-6.248c.194-.209.374-.429.54-.659l.13-.155Z"/>
            </svg>
        </div>
    </div>

    <div class="w-10/10 h-7/10 p-1">
        <div class="divider">活動手冊</div>
        <div class="carousel rounded-box w-10/10">
            <div v-for="(pageObj, page_i) in manualPages" class="carousel-item w-10/10 justify-center" :id="'page_' + page_i">
                <img :src="pageObj.src" :alt="pageObj.alt" class="w-fit" />
            </div>
        </div>
        <div class="flex w-10/10 justify-center gap-2 py-2">
            <a v-for="(pageObj, page_i) in manualPages" :href="'#page_' + page_i" class="btn btn-xs">{{ page_i + 1 }}</a>
        </div>
    </div>
</div>

</template>

<style scoped>

</style>
