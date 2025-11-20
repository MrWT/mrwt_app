<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String,
        user_role: String, 
    });

    onMounted(() => {
        console.log("Achievement mounted.");
        init();
    });

    let appState = ref("");
    // 未解鎖清單
    let nonAchieveEvents = reactive([]);
    // 已解鎖清單
    let achieveEvents = reactive([]);
    // 編輯物件
    let editObj_id = ref("");
    let editObj_description = ref("");
    let editObj_achieveTime = ref("");

    // 初始化 component
    function init(){
        console.log("Achievement.init");
        console.log("Achievement.props.title=", props.title);
        console.log("Achievement.props.account=", props.account);
        console.log("Achievement.props.user_role=", props.user_role);

        fetchInitData();
    }   
    // 取得初始資料
    function fetchInitData(){
        nonAchieveEvents.splice(0, nonAchieveEvents.length);
        achieveEvents.splice(0, achieveEvents.length);

        let fetchAchievementPromise = fetchData({
            api: "get_achievement",
            data: {
                account: props.account,
            }
        });
        Promise.all([fetchAchievementPromise]).then((values) => {
            console.log("fetchAchievementPromise.values=", values);

            values[0].forEach((aeObj, e_i) => {
                if(aeObj["achieve_time"]){
                    achieveEvents.push({
                        id: aeObj["id"],
                        description: aeObj["description"],
                        achieve_time: aeObj["achieve_time"],
                    });
                }else{
                    nonAchieveEvents.push({
                        id: aeObj["id"],
                        description: aeObj["description"],
                        achieve_time: aeObj["achieve_time"],
                    });
                }
            });

            nonAchieveEvents.sort((x, y) => {
                if(x["description"] > y["description"]) return 1;                
                if(x["description"] < y["description"]) return -1;                
                return 0;                
            });
            achieveEvents.sort((x, y) => {
                if(x["achieve_time"] > y["achieve_time"]) return -1;
                if(x["achieve_time"] < y["achieve_time"]) return 1;

                if(x["description"] > y["description"]) return 1;                
                if(x["description"] < y["description"]) return -1;                
                return 0;                
            });
        });
    }
    // 開啟 edit modal
    function openEditModal(selObj){
        document.getElementById("editModal").showModal();

        if(selObj.id){
            editObj_id.value = selObj.id;
            editObj_description.value = selObj.description;
            editObj_achieveTime.value = selObj.achieve_time;
        }else{
            editObj_id.value = "NEW";
            editObj_description.value = "";
            editObj_achieveTime.value = "";
        }

        console.log("openEditModal.editObj_id=" + editObj_id.value);
        console.log("openEditModal.editObj_description=" + editObj_description.value);
        console.log("openEditModal.editObj_achieveTime=" + editObj_achieveTime.value);
    }
    // 關閉 edit modal
    function closeEditModal(){
        document.getElementById("editModal").close();
    }
    // 儲存 edit 結果
    function saveEdit(){
        let api_name = editObj_id.value === "NEW" ? "new_achievement" : "upd_achievement";
        let doc_id = editObj_id.value === "NEW" ? "" : editObj_id.value;
        console.log("saveEdit.api_name=" + api_name);
        console.log("saveEdit.doc_id=" + doc_id);

        let editAchievementPromise = fetchData({
            api: api_name,
            data: {
                id: doc_id,
                account: props.account,
                description: editObj_description.value,
                achieve_time: editObj_achieveTime.value ?? "",
            }
        });
        Promise.all([editAchievementPromise]).then((values) => {
            console.log("editAchievementPromise.values=", values);

            fetchInitData();
            closeEditModal();
        });
    }
    
</script>

<template>

<div class="h-1/1 w-1/1 flex flex-col gap-2 overflow-y-auto">
    <!-- 待解鎖清單 -->
    <ul class="h-1/2 list bg-base-100 rounded-box shadow-md overflow-y-auto">
        <li class="p-4 pb-2 tracking-wide text-center font-semibold text-base bg-gray-300 text-2xl flex flex-row">
            待解鎖清單
        </li>
        <li v-for="(naObj, na_i) in nonAchieveEvents" class="list-row">
            <div class="text-4xl font-thin opacity-30 tabular-nums">{{ ((na_i+1) < 10 ? "0" : "") + (na_i+1) }}</div>
            <div class="list-col-grow">
                <div class="text-xl font-black">{{ naObj.description }}</div>
                <div class="text-base">{{ naObj.achieve_time }}</div>
            </div>
            <button class="btn btn-square bg-gray-300 border-0 border-black hover:border-2" @click="openEditModal(naObj)">
                <svg class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.779 17.779 4.36 19.918 6.5 13.5m4.279 4.279 8.364-8.643a3.027 3.027 0 0 0-2.14-5.165 3.03 3.03 0 0 0-2.14.886L6.5 13.5m4.279 4.279L6.499 13.5m2.14 2.14 6.213-6.504M12.75 7.04 17 11.28"/>
                </svg>
            </button>
        </li>
    </ul>
    <!-- 已解鎖清單 -->
    <ul class="h-1/2 list bg-base-100 rounded-box shadow-md overflow-y-auto">
        <li class="p-4 pb-2 tracking-wide text-center font-semibold text-base bg-emerald-200/50 text-2xl">
            已解鎖清單
        </li>
        <li v-for="(aeObj, ae_i) in achieveEvents" class="list-row">
            <div class="text-4xl font-thin opacity-30 tabular-nums">{{ ((ae_i+1) < 10 ? "0" : "") + (ae_i+1) }}</div>
            <div class="list-col-grow">
                <div class="text-xl font-black">{{ aeObj.description }}</div>
                <div class="text-base flex flex-row items-center">
                    <svg class="size-4 mr-2" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14v3m4-6V7a3 3 0 1 1 6 0v4M5 11h10a1 1 0 0 1 1 1v7a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1v-7a1 1 0 0 1 1-1Z"/>
                    </svg>
                    {{ aeObj.achieve_time }}
                </div>
            </div>
            <button class="btn btn-square bg-gray-300 border-0 border-black hover:border-2" @click="openEditModal(aeObj)">
                <svg class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.779 17.779 4.36 19.918 6.5 13.5m4.279 4.279 8.364-8.643a3.027 3.027 0 0 0-2.14-5.165 3.03 3.03 0 0 0-2.14.886L6.5 13.5m4.279 4.279L6.499 13.5m2.14 2.14 6.213-6.504M12.75 7.04 17 11.28"/>
                </svg>
            </button>
        </li>
    </ul>
</div>

<!-- 新增功能 -->
<button class="btn btn-square bg-yellow-300/50 border-0 border-black hover:border-2 absolute right-2 bottom-2 z-10" @click="openEditModal">
    <svg class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 12h14m-7 7V5"/>
    </svg>
</button>

<!-- edit modal -->
<dialog id="editModal" class="modal modal-end">
    <div class="modal-box h-1/2 w-1/1 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span v-if="editObj_id === 'NEW'" class="text-lg text-gray-900 text-center">新增解鎖項目</span>
            <span v-if="editObj_id !== 'NEW'" class="text-lg text-gray-900 text-center">編輯解鎖項目</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="h-1/1 w-1/1 flex flex-col overflow-y-auto gap-2 rounded-2xl">
            <div class="w-1/1 flex flex-row px-2 justify-start">
                <label class="label text-gray-900 w-1/1">
                    描述:
                    <input type="text" class="w-1/1" v-model="editObj_description" />
                </label>
            </div>
            <div class="w-1/1 flex flex-row px-2 justify-start">
                <label class="label text-gray-900 w-1/1">
                    達成日期
                    <input type="date" class="w-1/1" v-model="editObj_achieveTime" />
                </label>
            </div>
        </div>

        <div class="divider divider-primary"></div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 bg-gray-900 text-gray-100 hover:bg-yellow-100 hover:text-gray-900" @click.stop="closeEditModal">
                關閉
            </button>

            <button class="btn btn-ghost w-1/2 bg-gray-200 text-gray-900 hover:bg-yellow-100" @click.stop="saveEdit">
                儲存
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
