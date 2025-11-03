<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String,
        googleMapApiKey: String, 
    })

    onMounted(() => {
        console.log("Activity mounted.");
        init();
    });

    let appState = ref("");
    let selActivityMonth = ref("");
    let activityMonthList = reactive([]);
    let showActivity = ref(false);
    let activity = reactive({});
    let manualPages = reactive([]);

    // 初始化 component
    function init(){
        console.log("Activity.init");
        console.log("Activity.props.title=", props.title);
        console.log("Activity.props.account=", props.account);
        console.log("Activity.props.googleMapApiKey=", props.googleMapApiKey);

        get_activity_months();
    }   
    // 取得活動月份
    function get_activity_months(){
        let selMn = moment().format("YYYY-MM");

        let getActivityMonthPromise = fetchData({
            api: "get_activity_month",
        }, "KUO-FUNDS");
        Promise.all([getActivityMonthPromise]).then((values) => {
            console.log("getActivityMonthPromise.values=", values);
            // 先排序月份, 再建立清單
            values[0].sort((x, y) => {
                if(x > y) return 1;
                if(x < y) return -1;
                return 0;
            }).forEach((am, am_i) => {
                activityMonthList.push(am);
            });

            if(activityMonthList.length > 0){
                // 預設選擇當月份, 若當月份沒有在清單中, 則選擇接下來最接近的月份
                if(activityMonthList.indexOf(selMn) < 0){
                    for(let am_i = 0; am_i < activityMonthList.length; am_i++){
                        let am = activityMonthList[am_i];

                        if(selMn < am){
                            selMn = am;
                            break;
                        }
                    }
                }
                sel_activity_month( selMn );
            }
        });
    }
    // 依選擇的活動日期, 取得活動地點+活動說明
    function sel_activity_month(selMonth){
        console.log("sel_activity_month.selMonth=", selMonth);
        selActivityMonth.value = selMonth;
        get_activity();
    }
    // 取得活動內容
    function get_activity(){
        let getActivityPromise = fetchData({
            api: "get_activiy",
            data: {
                activity_month: selActivityMonth.value,
            },
        }, "KUO-FUNDS");
        Promise.all([getActivityPromise]).then((values) => {
            console.log("getActivityPromise.values=", values);
            activity = values[0][0];
            console.log("get_activity.activity=", activity);

            manualPages.splice(0, manualPages.length);
            if(activity){
                activity.pages.forEach((pageObj, page_i) => {
                    manualPages.push({
                        src: pageObj["page"],
                        alt: pageObj["memo"],
                    });
                });
            }

            // 決定是否呈現 activity
            showActivity.value = activity ? true : false;
        });
    }
    // 開啟 google map 方便設定導航
    function openGoogleMap(){
        const mapsUrl = `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent( activity.location_name )}`;
        window.open(mapsUrl, '_blank');
    }
    
</script>

<template>

<div class="w-1/1 overflow-x-auto overflow-y-hidden flex flex-row gap-2 shadow-3xl">
    <button v-for="(am, am_i) in activityMonthList" @click="sel_activity_month(am)"
        class="btn btn-ghost rounded-none border-0 border-b-2 hover:border-rose-500"
        :class="{'border-amber-600': am === selActivityMonth, 'border-gray-500': am !== selActivityMonth}">
        {{ am }}
    </button>
</div>

<div v-if="showActivity === true" class="w-10/10 h-9/10 flex flex-col gap-2 mt-2 overflow-y-auto">
    <div class="w-10/10 h-2/10 p-1">
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

    <div class="w-10/10 h-8/10 p-1">
        <div class="divider">活動說明</div>
        <div class="flex w-10/10 justify-center gap-2 py-2">
            <a v-for="(pageObj, page_i) in manualPages" :href="'#page_' + page_i" class="btn btn-xs">{{ page_i + 1 }}</a>
        </div>
        <div class="carousel rounded-box w-10/10">
            <div v-for="(pageObj, page_i) in manualPages" class="carousel-item w-10/10 justify-center" :id="'page_' + page_i">
                <img :src="pageObj.src" :alt="pageObj.alt" class="w-fit" />
            </div>
        </div>
    </div>
</div>

<div v-if="showActivity === false" class="w-10/10 h-9/10 flex flex-col gap-2 mt-5">
    <div class="w-1/1 h-1/2 justify-items-center">
        <svg class="size-full text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 15v3c0 .5523.44772 1 1 1h4v-4m-5 0v-4m0 4h5m-5-4V6c0-.55228.44772-1 1-1h16c.5523 0 1 .44772 1 1v1.98935M3 11h5v4m9.4708 4.1718-.8696-1.4388-2.8164-.235-2.573-4.2573 1.4873-2.8362 1.4441 2.3893c.3865.6396 1.2183.8447 1.8579.4582.6396-.3866.8447-1.2184.4582-1.858l-1.444-2.38925h3.1353l2.6101 4.27715-1.0713 2.5847.8695 1.4388"/>
        </svg>
    </div>
    <div class="w-1/1 text-center text-5xl">
        活動規劃中
    </div>
</div>

</template>

<style scoped>

</style>
