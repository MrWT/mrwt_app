<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("Member_KF mounted.");
        init();
    });

    let appState = ref("");
    let members = reactive([
        { index: "01", name: "name1", line_id: "line1", },
        { index: "02", name: "name2", line_id: "line2", },
        { index: "03", name: "name3", line_id: "line3", },
        { index: "04", name: "name4", line_id: "line4", },
        { index: "05", name: "name5", line_id: "line5", },
        { index: "06", name: "name6", line_id: "line6", },
        { index: "07", name: "name7", line_id: "line7", },
        { index: "08", name: "name8", line_id: "line8", },
        { index: "09", name: "name9", line_id: "line9", },
        { index: "10", name: "name10", line_id: "line10", },
        { index: "11", name: "name11", line_id: "line11", },
        { index: "12", name: "name12", line_id: "line12", },
        { index: "13", name: "name13", line_id: "line13", },
        { index: "14", name: "name14", line_id: "line14", },
        { index: "15", name: "name15", line_id: "line15", },
        { index: "16", name: "name16", line_id: "line16", },
        { index: "17", name: "name17", line_id: "line17", },
    ]);

    // 初始化 component
    function init(){
        console.log("Member_KF.init");
        console.log("Member_KF.props.title", props.title);
        console.log("Member_KF.props.account", props.account);
    }    
    
</script>

<template>

<div class="w-10/10 h-10/10 flex flex-col gap-2">
    <div class="w-10/10">
        <ul class="list bg-base-100 rounded-box shadow-md">
            <li v-for="(memObj, mem_i) in members" class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">{{ memObj.index }}</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M12 4a4 4 0 1 0 0 8 4 4 0 0 0 0-8Zm-2 9a4 4 0 0 0-4 4v1a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2v-1a4 4 0 0 0-4-4h-4Z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div>{{ memObj.name }}</div>
                    <div class="text-xs uppercase font-semibold opacity-60">Line ID: {{ memObj.line_id }}</div>
                </div>
            </li>
        </ul>
    </div>
</div>

</template>

<style scoped>

</style>
