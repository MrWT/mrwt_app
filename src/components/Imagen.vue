<script setup>
    import { ref, reactive, onMounted, onUpdated, watch, nextTick } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        title: String,
        account: String,
        user_role: String,
        imagenOption: Object,
    })

    onMounted(() => {
        console.log("imagen mounted.");
        init();
    });

    onUpdated(() => {
        console.log("imagen updated.");

    });

    let appState = ref("");
    
    let promptOptions = reactive({
        scene: [],
        specifyView: [],
        paintStyle: [],
    });

    let setPrompt = reactive({
        text: "",
        scene: "日常",
        specifyView: "無",
        paintStyle: "無",
    });

    let promptImg = reactive({
        state: "INIT",
        prompt: "",
        src: "",
    });

    // 初始化 component
    function init(){
        console.log("imagen.init");
        console.log("imagen.props.title", props.title);
        console.log("imagen.props.account", props.account);
        console.log("imagen.props.user_role", props.user_role);
        console.log("imagen.props.imagenOption", props.imagenOption);

        fetchInitData();
        combinePrompt();
    }
    // 取得初始資料
    function fetchInitData(){
        promptOptions.scene = props.imagenOption.scene.split(",");
        promptOptions.specifyView = props.imagenOption.specify_view.split(",");
        promptOptions.paintStyle = props.imagenOption.paint_style.split(",");
    }
    // 開始生成圖片
    function makePromptImg(){
        console.log("makePromptImg.prompt=" + promptImg.prompt);

        // 開啟 making 狀態
        {
            promptImg.state = "MAKING";
        }

        let promptImgPromise = fetchData({
            api: "ai_gen_image",
            data: {
                prompt: "幫我生成一張" + promptImg.prompt,
            }
        }, "AI");
        Promise.all([promptImgPromise]).then((values) => {
            console.log("promptImgPromise.values=", values);
            promptImg.src = values[0];
            // 關閉 making 狀態
            promptImg.state = "DONE";

            openImageModal();
        });
    }
    // 開啟 image modal
    function openImageModal(){
        document.getElementById("imageModal").showModal();
    }
    // 關閉 setting modal
    function closeImageModal(){
        document.getElementById("imageModal").close();
    }
    // 組合 prompt
    function combinePrompt(){
        let prompt = "";

        prompt += "幫我生成一張";
        prompt += "適用於'" + setPrompt.scene + "'情境的畫";
        if(setPrompt.specifyView !== "無"){
            prompt += ", 以'" + setPrompt.specifyView + "'為背景";
        }
        if(setPrompt.text){
            prompt += ", 並且另外在畫中題字'" + setPrompt.text + "' ";
        }
        if(setPrompt.paintStyle !== "無"){
            prompt += ", 以'" + setPrompt.paintStyle + "'畫風呈現這張畫";
        }

        promptImg.prompt = prompt;
    }
    // 下載圖片
    function downloadImage(){
        let a = document.createElement("a"); //Create <a>
        a.href = promptImg.src; //Image Base64 Goes here
        a.download = "長輩圖_" + moment().format("YYYYMMDD_HHmmss") + ".png"; //File name Here
        a.click(); //Downloaded file
    }


</script>

<template>

<div class="w-1/1 h-1/1 flex flex-col">
    <div class="w-1/1 flex flex-col">
        <span class="w-1/1 bg-rose-200/50 p-2">
            適用場景:
        </span>
        <div class="w-1/1 grid grid-cols-2 md:grid-cols-3 gap-1">
            <label v-for="(option, option_i) in promptOptions.scene" class="w-1/1">
                <input type="radio" :value="option" v-model="setPrompt.scene" @change="combinePrompt" />
                {{ option }}
            </label>
        </div>
    </div>
    <div class="w-1/1 flex flex-col">
        <span class="w-1/1 bg-rose-200/50 p-2">
            指定景色:
        </span>
        <div class="w-1/1 grid grid-cols-2 md:grid-cols-3 gap-1">
            <label v-for="(option, option_i) in promptOptions.specifyView" class="w-1/1">
                <input type="radio" :value="option" v-model="setPrompt.specifyView" @change="combinePrompt" />
                {{ option }}
            </label>
        </div>
    </div>
    <div class="w-1/1 flex flex-col">
        <span class="w-1/1 bg-rose-200/50 p-2">
            指定畫風:
        </span>
        <div class="w-1/1 grid grid-cols-2 md:grid-cols-3 gap-1">
            <label v-for="(option, option_i) in promptOptions.paintStyle" class="w-1/1">
                <input type="radio" :value="option" v-model="setPrompt.paintStyle" @change="combinePrompt" />
                {{ option }}
            </label>
        </div>
    </div>
    <div class="w-1/1 flex flex-col">
        <span class="w-1/1 bg-rose-200/50 p-2">
            呈現於圖片中的文字:
        </span>
        <div class="w-1/1">
            <input type="text" placeholder="e.g., 早安" class="input w-1/1" v-model="setPrompt.text" @keyup.stop="combinePrompt" />
        </div>
    </div>

    <div class="w-1/1 flex flex-col">
        <span class="w-1/1 bg-rose-200/50 p-2">
            預覽 Prompt:
        </span>
        <div class="w-1/1 h-fit">
            <textarea class="border rounded-xl textarea w-1/1 h-1/1" disabled v-model="promptImg.prompt"></textarea>                
        </div>
    </div>
    
    <div class="divider divider-primary"></div>
    <button v-if="promptImg.state !== 'MAKING'" class="btn btn-square bg-black text-white w-1/1" @click="makePromptImg">
        生成
    </button>
</div>

<!-- image modal -->
<dialog id="imageModal" class="modal">
    <div class="modal-box h-7/10 w-8/10 flex flex-col bg-neutral-100">
        <div class="flex flex-col justify-center">
            <span class="text-xl text-gray-900 text-center">您的長輩圖</span>
            <div class="divider divider-primary"></div>
        </div>
        <div class="h-1/1 w-1/1 flex flex-col overflow-y-auto">
            <div v-if="promptImg.state === 'INIT'" class="h-1/1 w-1/1 bg-gray-200"></div>

            <div v-if="promptImg.state === 'MAKING'" class="skeleton h-1/1 w-1/1"></div>

            <img v-if="promptImg.state === 'DONE'" :src="promptImg.src" />
        </div>
        <div class="divider divider-primary"></div>
        <div class="modal-action">
            <button class="btn btn-ghost w-1/2 bg-gray-200 text-gray-900 hover:bg-yellow-100" @click="closeImageModal">
                <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18 17.94 6M18 18 6.06 6"/>
                </svg>
                關閉預覽
            </button>

            <button class="btn btn-ghost w-1/2 bg-gray-200 text-gray-900 hover:bg-yellow-100" @click="downloadImage">
                <svg class="size-8" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 13V4M7 14H5a1 1 0 0 0-1 1v4a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1v-4a1 1 0 0 0-1-1h-2m-1-5-4 5-4-5m9 8h.01"/>
                </svg>
                儲存設定
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
