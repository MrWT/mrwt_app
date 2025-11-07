<script setup>
    import { ref, reactive, onMounted, onUpdated, watch } from 'vue'
    import moment from 'moment'
    import { gsap } from "gsap"
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

    onUpdated(() => {
        console.log("Recollection updated.");

        // 檢查 iframe 是否已將 youtube 載入完成
        let iframe_el = document.querySelectorAll("iframe");
        iframe_el.forEach((el, el_i) => {
            let iframe_id = el.id;
            let div_id = "div_" + iframe_id.substring(7);

            // 讓 skeleton 閃爍
            let tl = gsap.timeline({ yoyo: true, repeat: -1 });
            tl.to("#" + div_id, {
                duration: 0.3,
                opacity: 0,
            });
            // 當 iframe 已載入完成 youtube 
            el.addEventListener("load", function() {
                //console.log("Iframe content has loaded!", el.id);

                //console.log("iframe_id=" + iframe_id);
                //console.log("div_id=" + div_id);

                document.getElementById(div_id).classList.add("hidden");
                el.classList.remove("hidden");
            });
        });
    });

    let appState = ref("");
    let dataList = reactive([]);
    let selectTagList = reactive([]);
    let selDate = ref( moment().format("YYYY-MM") );
    let selTag = ref("All");

    let editTagList = reactive([]);
    let editID = ref("");
    let editDate = ref( moment().format("YYYY-MM-DD") );
    let editTag = ref("");
    let editTags = reactive([]);
    let editType = ref("youtube");
    let editMemo = ref("");
    let editSrc = ref("");

    // 初始化 component
    function init(){
        console.log("Recollection.init");
        console.log("Recollection.props.title", props.title);
        console.log("Recollection.props.user_role", props.user_role);
        
        // 取得初始資料
        getInitData();
        // 取得回憶清單
        getRecollectionList();        
    }     
    // 取得初始資料
    function getInitData(){
        selectTagList.splice(0, selectTagList.length);
        editTagList.splice(0, editTagList.length);

        // 取得標籤清單
        let getRecollectionTagsPromise = fetchData({
            api: "get_recollection_tags",
        });
        Promise.all([getRecollectionTagsPromise]).then((values) => {
            console.log("getRecollectionTagsPromise.values=", values);

            selectTagList.push( "All" );
            values[0].forEach((tag, tag_i) => {
                selectTagList.push( tag );
                editTagList.push( tag );
            });
        });
    }
    // 取得回憶清單
    function getRecollectionList(){
        console.log("getRecollectionList.selDate=", selDate.value);
        console.log("getRecollectionList.selTag=", selTag.value);

        // 清空 dataList
        dataList.splice(0, dataList.length);

        // 取得回憶清單
        let getRecollectionPromise = fetchData({
            api: "get_recollection",
            data: {
                date: selDate.value,
                tag: selTag.value,
            }
        });
        Promise.all([getRecollectionPromise]).then((values) => {
            console.log("getRecollectionPromise.values=", values);

            values[0].forEach((recObj, rec_i) => {
                // 組合 video src
                let src_of_dataObj = "";
                {
                    if(recObj["src"].indexOf("https://") >= 0){
                        src_of_dataObj = recObj["src"];
                    }else{
                        src_of_dataObj = "https://www.youtube.com/embed/" + recObj["src"];
                    }
                    src_of_dataObj += "?autoplay=0";
                }

                dataList.push({
                    id: recObj["id"],
                    date: recObj["date"],
                    tags: recObj["tags"].split(","),
                    src: src_of_dataObj,
                    memo: recObj["memo"],
                    type: recObj["type"], 

                    divID: "div_" + recObj["id"],
                    iframeID: "iframe_" + recObj["id"],
                });
            });
        });
    }
    // 開啟 editModal
    function openEditModal(editObj){
        if(editObj.hasOwnProperty("id")){
            editID.value = editObj["id"];
            editDate.value = editObj["date"];
            
            editTags.splice(0, editTags.length);
            editObj["tags"].forEach((tag, tag_i) => {
                editTags.push(tag);
            });
            editTag.value = "";
            
            editType.value = editObj["type"];
            editMemo.value = editObj["memo"];
            editSrc.value = editObj["src"];
        }else{
            editID.value = "NEW";
            editDate.value = moment().format("YYYY-MM-DD");
            editTags.splice(0, editTags.length);
            editTag.value = "";
            editType.value = "youtube";
            editMemo.value = "";
            editSrc.value = "";
        }

        //console.log("openEditModal.editID=", editID.value);

        document.getElementById("editModal").showModal();
    }
    // 儲存一則回憶
    function saveEdit(){
        //console.log("saveEdit", editID.value ?? "NEW");

        let editRecollectionPromise = fetchData({
            api: "edit_recollection",
            data: {
                id: editID.value,
                date: editDate.value,
                memo: editMemo.value,
                tags: editTags.join(","),
                type: editType.value,
                src: editSrc.value,
            }
        });
        Promise.all([editRecollectionPromise]).then((values) => {
            console.log("editRecollectionPromise.values=", values);

            getInitData();
            getRecollectionList();
            closeEditModal();
        });
    }
    // 移除一個 tag
    function eraseEditTag(tag){
        let tag_i = editTags.indexOf(tag);
        if(tag_i >= 0){
            editTags.splice(tag_i, 1);
        }
    }
    // 關閉 editModal
    function closeEditModal(){
        document.getElementById("editModal").close();
    }

    // 監視
    watch(selTag, (newTag, oldTag) => {
        //console.log("watch.newTag=", newTag);
        //console.log("watch.oldTag=", oldTag);

        // 取得回憶清單
        getRecollectionList();
    });
    watch(selDate, (newDate, oldDate) => {
        //console.log("watch.newDate=", newDate);
        //console.log("watch.oldDate=", oldDate);

        if(selTag.value !== "All"){
            selTag.value = "All";
        }else{
            // 取得回憶清單
            getRecollectionList();
        }
    });
    watch(editTag, (newTag, oldTag) => {
        //console.log("watch.newTag=", newTag);
        //console.log("watch.oldTag=", oldTag);

        if(editTags.indexOf(newTag) < 0){
            editTags.push(newTag);
        }
    });

</script>

<template>

<div class="w-1/1 h-1/5 flex flex-row gap-1 bg-gray-200 p-2 rounded-2xl">
    <div class="flex flex-row gap-1 w-2/3">
        <label class="text-md w-2/3 md:w-1/2">
            月份
            <input type="month" class="input" v-model="selDate" />
        </label>

        <label class="text-md w-1/3 md:w-1/2">
            標籤
            <select class="select" v-model="selTag">
                <option disabled selected>選擇一種標籤</option>
                <option v-for="(tag, tag_i) in selectTagList" :value="tag">{{ tag }}</option>
            </select>
        </label>
    </div>
    <div class="flex flex-row w-1/3 justify-end">
        <button class="btn h-1/1" @click="openEditModal">新增</button>
    </div>
</div>

<div class="w-1/1 h-4/5 overflow-y-auto overflow-x-hidden">

    <ul v-if="dataList.length > 0" class="timeline timeline-snap-icon max-md:timeline-compact timeline-vertical">
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
                <time class="font-mono italic">{{ dataObj.date }}</time>
                <div class="w-1/1 flex flex-row gap-2">
                    <div v-for="(tag, tag_i) in dataObj.tags" class="badge badge-md badge-soft badge-info">
                        {{ tag }}
                    </div>
                </div>
                <div class="text-xl flex flex-row items-center">
                    {{ dataObj.memo }}
                    <button class="btn btn-ghost" @click="openEditModal(dataObj)">
                        <svg class="size-4 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                            <path fill-rule="evenodd" d="M14 4.182A4.136 4.136 0 0 1 16.9 3c1.087 0 2.13.425 2.899 1.182A4.01 4.01 0 0 1 21 7.037c0 1.068-.43 2.092-1.194 2.849L18.5 11.214l-5.8-5.71 1.287-1.31.012-.012Zm-2.717 2.763L6.186 12.13l2.175 2.141 5.063-5.218-2.141-2.108Zm-6.25 6.886-1.98 5.849a.992.992 0 0 0 .245 1.026 1.03 1.03 0 0 0 1.043.242L10.282 19l-5.25-5.168Zm6.954 4.01 5.096-5.186-2.218-2.183-5.063 5.218 2.185 2.15Z" clip-rule="evenodd"/>
                        </svg>
                    </button>
                </div>
                <div :id="dataObj.divID" class="w-[300px] h-[150px] md:w-[600px] md:h-[300px] mt-2">
                    <div class="skeleton h-1/1 w-1/1"></div>
                </div>
                <iframe :id="dataObj.iframeID" class="w-fit md:w-[600px] md:h-[300px] mt-2 hidden" :src="dataObj.src" frameborder="0"></iframe>
            </div>
            <hr />
        </li>
    </ul>

    <div v-if="dataList.length === 0" class="w-1/1 text-center text-5xl mt-2">
        查無資料
    </div>
</div>

<!-- editModal -->
<dialog id="editModal" class="modal modal-end">
    <div class="modal-box h-1/1 md:w-1/3 w-2/3 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span v-if="editID === 'NEW'" class="text-lg text-gray-900 text-center">新增一則回憶</span>
            <span v-if="editID !== 'NEW'" class="text-lg text-gray-900 text-center">調整一則回憶</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="h-4/5 w-1/1 flex flex-col overflow-y-auto gap-3 border rounded-2xl p-2">
            <label class="text-md w-1/1">
                日期
                <input type="date" class="input" v-model="editDate" />
            </label>

            <label class="text-md w-1/1">
                標籤
                <select class="select" v-model="editTag">
                    <option disabled selected>選擇一種標籤</option>
                    <option v-for="(tag, tag_i) in editTagList" :value="tag">{{ tag }}</option>
                </select>
            </label>
            <div class="w-1/1 flex flex-row gap-2">
                <div v-for="(tag, tag_i) in editTags" class="badge badge-md badge-soft badge-info">
                    {{ tag }}
                    <button class="btn btn-xs btn-ghost" @click="eraseEditTag(tag)">
                        <svg class="size-6 text-red-500" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                            <path fill-rule="evenodd" d="M2 12C2 6.477 6.477 2 12 2s10 4.477 10 10-4.477 10-10 10S2 17.523 2 12Zm7.707-3.707a1 1 0 0 0-1.414 1.414L10.586 12l-2.293 2.293a1 1 0 1 0 1.414 1.414L12 13.414l2.293 2.293a1 1 0 0 0 1.414-1.414L13.414 12l2.293-2.293a1 1 0 0 0-1.414-1.414L12 10.586 9.707 8.293Z" clip-rule="evenodd"/>
                        </svg>
                    </button>
                </div>
            </div>

            <label class="text-md w-1/1">
                說明
                <input type="text" class="input" v-model="editMemo" />
            </label>

            <label class="text-md w-1/1">
                來源(YouTube ID): <input type="text" class="input w-1/3" v-model="editSrc" />
            </label>

            <label class="text-md w-1/1">
                類型
                <select class="select" v-model="editType">
                    <option disabled selected>選擇一種類型</option>
                    <option value="youtube">youtube</option>
                </select>
            </label>
        </div>

        <div class="divider divider-primary"></div>
        <div class="modal-action">
            <button class="btn btn-ghost w-5/10 bg-gray-200 text-gray-900 hover:bg-yellow-100" @click.stop="closeEditModal">
                關閉
            </button>

            <button class="btn btn-ghost w-5/10 bg-gray-200 text-gray-900 hover:bg-yellow-100" @click.stop="saveEdit">
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
