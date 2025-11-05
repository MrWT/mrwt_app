<script setup>
    import { ref, reactive, onMounted, watch } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        user_role: String,
    })

    onMounted(() => {
        console.log("Recollection mounted.");
        init();
    });

    let isBuilding = ref(true);
    let appState = ref("");
    let dataList = reactive([]);

    // 初始化 component
    function init(){
        console.log("Recollection.init");
        console.log("Recollection.props.title", props.title);

        isBuilding.value = props.user_role === "admin" ? false : true;

        getRecollectionList();
    }     

    function getRecollectionList(){

        let getRecollectionPromise = fetchData({
            api: "get_recollection",
        });
        Promise.all([getRecollectionPromise]).then((values) => {
            console.log("getRecollectionPromise.values=", values);
        });
    }

</script>

<template>

<div v-if="isBuilding === false"  class="w-10/10">
    <div class="text-lg"></div>
</div>

<div v-if="isBuilding === false"  class="w-10/10">

    <ul class="timeline timeline-snap-icon max-md:timeline-compact timeline-vertical">
        <li v-for="(dataObj, data_i) in dataList">
            <hr />
            <div class="timeline-middle">
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    viewBox="0 0 20 20"
                    fill="currentColor"
                    class="h-5 w-5"
                >
                    <path
                    fill-rule="evenodd"
                    d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.857-9.809a.75.75 0 00-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 10-1.06 1.061l2.5 2.5a.75.75 0 001.137-.089l4-5.5z"
                    clip-rule="evenodd"
                    />
                </svg>
            </div>
            <div :class="{'timeline-start mb-10 md:text-end': data_i % 2 === 0, 'timeline-end md:mb-10': data_i % 2 === 1}">
                <time class="font-mono italic">{{ dataObj.time }}</time>
                <div class="text-lg font-black">{{ dataObj.title }}</div>
                {{ dataObj.content }}
            </div>
            <hr />
        </li>
    </ul>

</div>

<div v-if="isBuilding === true" class="w-10/10 h-9/10 flex flex-col gap-2 mt-5">
    <div class="w-1/1 h-1/2 justify-items-center">
        <svg class="size-full text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 15v3c0 .5523.44772 1 1 1h4v-4m-5 0v-4m0 4h5m-5-4V6c0-.55228.44772-1 1-1h16c.5523 0 1 .44772 1 1v1.98935M3 11h5v4m9.4708 4.1718-.8696-1.4388-2.8164-.235-2.573-4.2573 1.4873-2.8362 1.4441 2.3893c.3865.6396 1.2183.8447 1.8579.4582.6396-.3866.8447-1.2184.4582-1.858l-1.444-2.38925h3.1353l2.6101 4.27715-1.0713 2.5847.8695 1.4388"/>
        </svg>
    </div>
    <div class="w-1/1 text-center text-5xl">
        功能規劃中
    </div>
</div>

</template>

<style scoped>

</style>
