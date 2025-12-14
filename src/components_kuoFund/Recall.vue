<script setup>
    import { ref, reactive, onMounted, watch } from 'vue'
    import { fetchData } from "@/composables/fetchData"
    import { GoogleMap, AdvancedMarker, InfoWindow, Polyline } from 'vue3-google-map'

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
    })

    onMounted(() => {
        console.log("Recall mounted.");
        init();
    });


    let appState = ref("");
    // 回憶清單
    let recallList = reactive([]);
    // 選擇要查閱的回憶
    let selObj_id = ref("");
    let selObj_content = ref("");
    
    // 初始化 component
    function init(){
        console.log("Recall.init");
        console.log("Recall.props.title", props.title);
        console.log("Recall.props.account", props.account);

        fetchInitData();
    }
    // 取得初始資料
    function fetchInitData(){
        // 清空回憶清單
        recallList.splice(0, recallList.length);
        // 取得回憶清單
        let fetchPromise_recall = fetchData({
            api: "get_recall",
        }, "KUO-FUNDS");
        Promise.all([fetchPromise_recall]).then((values) => {
            console.log("fetchInitData.values=", values);

            values[0].forEach((recallObj, r_i) => {
                recallList.push( recallObj );
            });
        });
    }
    // 開啟 Remove 再確認 modal
    function openRemoveConfirmModal(recallObj){
        console.log("openRemoveConfirmModal.recallObj=", recallObj);

        selObj_id.value = recallObj.doc_id;
        selObj_content.value = recallObj.content;
        console.log("openRemoveConfirmModal.selObj_id=", selObj_id.value);
        console.log("openRemoveConfirmModal.selObj_content=", selObj_content.value);

        document.getElementById("removeConfirmModal").showModal();
    }
    // 關閉 Remove 再確認 modal
    function closeRemoveConfirmModal(){
        document.getElementById("removeConfirmModal").close();
    }
    // 刪除回憶
    function removeRecall(){
        console.log("removeRecall.selRemove_id=", selObj_id.value);

        editRecall("DELETE");
    }
    // 開啟 detail modal
    function openDetailModal(recallObj){
        console.log("openDetailModal.recallObj=", recallObj);

        if(recallObj.hasOwnProperty("doc_id")){
            selObj_id.value = recallObj.doc_id;
            selObj_content.value = recallObj.content;
        }else{
            selObj_id.value = "NEW";
            selObj_content.value = "";
        }

        document.getElementById("detailModal").showModal();
    }
    // 關閉 detail modal
    function closeDetailModal(){
        document.getElementById("detailModal").close();
    }
    // 更新回憶
    function editRecall(opMode){
        console.log("editRecall.selObj_id=", selObj_id.value);

        opMode = selObj_id.value === "NEW" ? "NEW" : opMode;

        let editPromise_recall = fetchData({
            api: "edit_recall",
            data: {
                doc_id: selObj_id.value,
                content: selObj_content.value,
                active: opMode === "DELETE" ? "N": "Y",
            },
        }, "KUO-FUNDS");
        Promise.all([editPromise_recall]).then((values) => {
            console.log("editPromise_recall.values=", values);
            // 更新回憶清單
            fetchInitData();

            let opObj = {
                message: "",
                status: true,
            };
            opObj.status = values[0]["result"];
            if(values[0]["result"] === true){
                if(opMode === "DELETE"){
                    opObj.message = "刪除成功";
                }else if(opMode === "EDIT"){
                    opObj.message = "編輯成功";
                }else{
                    opObj.message = "新增成功";
                }
            }else{
                opObj.message = "Error: " + values[0]["message"];
            }
            // 將 message 傳給 App.vue 
            emit('popupMessage', opObj.status, opObj.message); // Emitting the event with data

            // 關閉 remove 再確認 modal
            closeRemoveConfirmModal();
            // 關閉 detail modal
            closeDetailModal();
        });
    }

</script>

<template>

<div class="w-1/1 h-1/8 flex justify-end">
    <button class="btn btn-square btn-ghost text-gray-900" title="新增回憶" @click="openDetailModal">
        <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 7.757v8.486M7.757 12h8.486M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
        </svg>

    </button>
</div>

<div class="w-1/1 h-7/8 overflow-y-auto">
    <ul v-if="recallList.length > 0" class="list rounded-box shadow-md">
        <li v-for="(rObj, r_i) in recallList" class="list-row hover:bg-yellow-100">
            <div class="text-4xl font-thin opacity-30 tabular-nums">{{ ((r_i + 1) < 10 ? "0" : "") + (r_i + 1) }}</div>
            <div class="list-col-grow">
                <div class="text-lg">
                    {{ rObj.content }}
                </div>
            </div>
            <button class="btn btn-square btn-ghost text-red-900" title="刪除回憶" @click="openRemoveConfirmModal(rObj)">
                <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m15 9-6 6m0-6 6 6m6-3a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
                </svg>
            </button>
            <button class="btn btn-square btn-ghost" title="回憶內容" @click="openDetailModal(rObj)">
                <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 14H4m6.5 3L8 20m5.5-3 2.5 3M4.88889 17H19.1111c.4909 0 .8889-.4157.8889-.9286V4.92857C20 4.41574 19.602 4 19.1111 4H4.88889C4.39797 4 4 4.41574 4 4.92857V16.0714c0 .5129.39797.9286.88889.9286ZM13 14v-3h4v3h-4Z"/>
                </svg>
            </button>
        </li>
    </ul>
    <div class="w-1/1 text-center bg-gray-300">
        <span class="text-gray-900 font-black text-3xl underline" v-if="recallList.length === 0">查無資料</span>
    </div>
</div>

<!-- recall detail modal -->
<dialog id="detailModal" class="modal">
    <div class="modal-box h-4/5 w-1/1 max-w-3xl flex flex-col bg-neutral-100">
        <div class="h-1/1 w-1/1 text-center text-gray-900 font-black p-2">
            <span class="text-2xl">
                <span v-if="selObj_id === 'NEW'">{{ "新增" }}</span>
                <span v-if="selObj_id !== 'NEW'">{{ "編輯" }}</span>
                回憶內容
            </span>
            <div class="divider divider-primary"></div>

            <textarea class="textarea w-1/1 h-4/5" v-model="selObj_content">{{ selObj_content }}</textarea>
        </div>

        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 bg-gray-900 text-gray-100 hover:underline hover:bg-gray-100 hover:text-black" @click="closeDetailModal">
                <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m15 9-6 6m0-6 6 6m6-3a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
                </svg>
                關閉
            </button>

            <button class="btn btn-ghost w-1/2 bg-gray-200 text-gray-900 hover:underline hover:bg-yellow-300 hover:text-black" @click="editRecall('EDIT')">
                <svg class="size-6" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linejoin="round" stroke-width="2" d="M10 3v4a1 1 0 0 1-1 1H5m14-4v16a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V7.914a1 1 0 0 1 .293-.707l3.914-3.914A1 1 0 0 1 9.914 3H18a1 1 0 0 1 1 1Z"/>
                </svg>
                儲存
            </button>
        </div>
    </div>
    <form method="dialog" class="modal-backdrop">
        <button>close</button>
    </form>
</dialog>

<!-- remove recall modal -->
<dialog id="removeConfirmModal" class="modal">
    <div class="modal-box h-5/10 w-10/10 flex flex-col bg-neutral-500">
        <div class="h-2/10 w-10/10 text-center text-gray-100 font-black">
            <span class="text-2xl">刪除前, 再跟您確認一次~</span>
            <div class="divider divider-error"></div>
        </div>
        <div class="h-5/10 w-1/1 text-gray-900 p-2">
            <textarea class="textarea w-1/1 h-1/1" readonly>{{ selObj_content }}</textarea>
        </div>
        <div class="divider divider-error"></div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 text-gray-100 hover:bg-gray-100 hover:text-gray-900 hover:underline" @click="closeRemoveConfirmModal">
                取消
            </button>

            <button class="btn btn-ghost w-1/2 text-gray-100 hover:bg-gray-100 hover:text-gray-900 hover:underline" @click="removeRecall">
                確認
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
