<script setup>
    import { ref, reactive, onMounted, onUpdated, watch, nextTick } from 'vue'
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
        console.log("imagen mounted.");
        init();
    });

    onUpdated(() => {
        console.log("imagen updated.");

    });

    let appState = ref("");
    let userMessage = ref("");
 
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

        fetchInitData();
    }
    // 取得初始資料
    function fetchInitData(){
        
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
        });
    }



</script>

<template>

<div class="w-1/1 h-1/1">
    <div class="w-1/1 h-1/3 flex flex-col">
        預覽:
        <textarea class="border rounded-xl textarea w-1/1" :disabled="promptImg.state === 'MAKING'" v-model="promptImg.prompt"></textarea>                
    </div>            

    <div class="w-1/1 h-1/2 justify-items-center mt-2 border rounded-xl">
        <div v-if="promptImg.state === 'INIT'" class="h-1/1 w-1/1 bg-gray-200"></div>

        <div v-if="promptImg.state === 'MAKING'" class="skeleton h-1/1 w-1/1"></div>

        <img v-if="promptImg.state === 'DONE'" :src="promptImg.src" />
    </div>

    <button v-if="promptImg.state !== 'MAKING'" class="btn btn-square bg-black text-white w-1/2" @click="makePromptImg">
        生成
    </button>

</div>


</template>

<style scoped>

</style>
