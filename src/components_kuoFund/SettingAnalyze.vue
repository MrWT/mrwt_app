<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("SettingAnalyze mounted.");
        init();
    });

    let appState = ref("");
    let analyzeResult = ref("");

    // 初始化 component
    function init(){
        console.log("SettingAnalyze.init");
        console.log("SettingAnalyze.props.title", props.title);
        console.log("SettingAnalyze.props.account", props.account);

        analyze_survey();
    }
    // ask ai analyze survey data
    function analyze_survey(){
        console.log("analyze_survey");

        let analyzePromise = fetchData({
            api: "analyze_survey",
        });
        Promise.all([analyzePromise]).then((values) => {
            console.log("analyzePromise.values=", values);

            analyzeResult.value = values[0].message;
        });

    } 

</script>

<template>

<div class="relative h-10/10 w-10/10 flex flex-col overflow-y-auto p-2">
    <button class="btn btn-ghost absolute top-0 right-0 z-10" @click="analyze_survey">
        <svg class="size-6 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.651 7.65a7.131 7.131 0 0 0-12.68 3.15M18.001 4v4h-4m-7.652 8.35a7.13 7.13 0 0 0 12.68-3.15M6 20v-4h4"/>
        </svg>
    </button>
    <p class="w-10/10 h-10/10" style="white-space:pre-wrap;">
        {{ analyzeResult }}
    </p>
</div>

</template>

<style scoped>

</style>
