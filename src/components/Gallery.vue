<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    import { gsap } from "gsap"
    import { DrawSVGPlugin } from "gsap/DrawSVGPlugin"
    import { MotionPathPlugin } from 'gsap/MotionPathPlugin'
    import { getRandomNumber } from "@/composables/random"

    const props = defineProps({
        title: String,
        account: String,
        cname: String,
        user_role: String,
        focus_news_topic: String,
    })

    onMounted(() => {
        console.log("gallery.mounted");
        init();

        setTimeout(() => {
            window.dispatchEvent(new Event('resize'));
        }, 1000);

    });

    let appState = ref("");
    let yesterday = ref("");
    let topicList = reactive([]);
    let topicBgColorCount = ref(4);
    let newsList = reactive({});

    // 初始化 component
    function init(){
        console.log("gallery.init");
        console.log("gallery.props.title=", props.title);
        console.log("gallery.props.account=", props.account);
        console.log("gallery.props.cname=", props.cname);
        console.log("gallery.props.user_role=", props.user_role);
        console.log("gallery.props.focus_news_topic=", props.focus_news_topic);
        appState.value = props.account === "KUOFAMILY" || props.user_role === "admin_kf" ? "k_funds" : "normal";

        yesterday.value = moment().add(-1, "d").format("YYYY-MM-DD");

        if(appState.value === "k_funds"){
            initGsap();
        }else{
            fetchInitData();
        }
    }    
    // 初始化 - 動畫效果
    function initGsap(){
        gsap.registerPlugin(DrawSVGPlugin);
        gsap.registerPlugin(MotionPathPlugin);

        // 候選顏色
        let candidateColors = ["green", "blue", "purple", "gold", "peru", "blanchedalmond", "blueviolet", "goldenrod"];
        let durations = [];
        for(let d_i = 0; d_i < 10; d_i++){
            durations.push( getRandomNumber(10, 20)/10 );
        }

        setTimeout(() => {
            let tl_back = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });
            let tl_word = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });

            for(let c_i = 0; c_i < candidateColors.length; c_i++){
                // 郭字背景
                tl_back.to("#" + "kuoBack", {
                    id: "tween_" + "kuoBack",
                    duration: getRandomNumber(30, 40)/10,
                    background: candidateColors[c_i],
                    ease: "bounce.inOut",
                });

                // 郭字
                tl_word.to("#" + "kuoWord", {
                    id: "tween_" + "kuoWord",
                    duration: getRandomNumber(20, 30)/10,
                    color: candidateColors[ candidateColors.length - 1 - c_i],
                    //ease: "bounce.inOut",
                });
            }

            // 玩 SVG
            let tl_path = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });
            let tl_circle = gsap.timeline({ yoyo: true, repeat: -1, repeatDelay: 3, });
            {
                tl_path.from("#myPath", {
                    drawSVG: false, 
                    duration: 2, 
                    repeat: -1,
                    repeatDelay: 3,
                    yoyo: true,
                    ease: "power1.inOut"
                });
                for(let c_i = 1; c_i <= 5; c_i++){
                    tl_circle.to("#myCircle" + c_i, {
                        duration: 2, 
                        repeat: -1,
                        repeatDelay: 3,
                        yoyo: true,
                        ease: "power1.inOut",
                        motionPath:{
                            path: "#myPath",
                            align: "#myPath",
                            autoRotate: true,
                            alignOrigin: [0.5, 0.5]
                        }
                    });
                }
            }
        }, 100);
    }
    // 取得初始資料 - 取得新聞資訊
    function fetchInitData(){
        // 清空資料
        {
            for (const prop in newsList) {
                if (Object.prototype.hasOwnProperty.call(newsList, prop)) {
                    delete newsList[prop];
                }
            }
            topicList.splice(0, topicList.length);
        }
        
        // 取得新聞資訊
        let fetchNewsPromise = fetchData({
            api: "get_news",
            data: {
                focus_topic: props.focus_news_topic ?? "All",
            }
        });
        Promise.all([fetchNewsPromise]).then((values) => {
            console.log("fetchNewsPromise.values=", values);

            values[0].forEach((newsObj, news_i) => {
                topicList.push({
                    desc: newsObj["desc"],
                    key: newsObj["key"],
                    tags: newsObj["tag"].split(","),
                });


                let contentList = [];
                try{
                    let content = newsObj["content"];
                    content = content.replace(/```json/g, "");
                    content = content.replace(/```/g, "");

                    //console.log("before parse.content=", content);
                    contentList = JSON.parse(" " + content + " ");
                }catch(ex){
                    console.log("newsObj.key=", newsObj.key);
                    console.log("newsObj.content=", newsObj.content);
                    console.error("JSON.parse(content).ex=", ex);
                }
                newsList[newsObj["key"]] = contentList;
            });

            console.log("newsList=", newsList);
        });
    }
    // 請 AI 重新取得指定 topic 的新聞資料
    function refetchSpecifyTopic(selTopicObj){
        console.log("refetchSpecifyTopic.selTopicObj=", selTopicObj);

        // 請 AI 重新取得指定 topic 的新聞資料
        let refetchPromise = fetchData({
            api: "fetch_daily_news",
            data: {
                specify_topic: selTopicObj["key"],
            }
        }, "AI");
        Promise.all([refetchPromise]).then((values) => {
            console.log("refetchPromise.values=", values);

            fetchInitData();
        });
    }

</script>

<template>
    <!-- 一般使用者 -->
    <div v-if="appState === 'normal'" class="text-center text-3xl w-1/1 bg-violet-200 rounded-2xl mb-2">{{ yesterday }} 關注新聞整理</div>
    <div v-if="appState === 'normal'" class="w-1/1 h-11/12 overflow-y-auto">
        <ul v-for="(topicObj, topic_i) in topicList" class="list bg-base-100 rounded-box shadow-2xl">
            <li class="p-4 pb-2 tracking-wide flex flex-row items-end text-gray-900 rounded-xl shadow-2xl z-10" 
                :class="{
                    'bg-lime-200/80': topic_i % topicBgColorCount === 0, 
                    'bg-amber-200/80': topic_i % topicBgColorCount === 1, 
                    'bg-indigo-200/80': topic_i % topicBgColorCount === 2, 
                    'bg-fuchsia-200/80': topic_i % topicBgColorCount === 3, 
                }"                
                style="position: sticky; top:0;">
                <div class="flex-1 flex flex-row items-end">
                    <div class="font-black text-xl mr-5">{{ topicObj.desc }}</div> 
                    <div v-for="(tag, tag_i) in topicObj.tags" class="mr-2 text-blue-900">{{ "#" + tag }}</div>
                </div>
                <div class="flex-none">
                    <button class="btn btn-ghost hover:bg-transparent" title="重新下載" @click="refetchSpecifyTopic(topicObj)">
                        <svg class="size-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 15v2a3 3 0 0 0 3 3h10a3 3 0 0 0 3-3v-2m-8 1V4m0 12-4-4m4 4 4-4"/>
                        </svg>
                    </button>
                </div>
            </li>
            <li v-if="newsList[topicObj.key].length === 0" class="list-row">
                <div class="list-col-grow">
                    <div class="text-black text-lg text-center">{{ "AI: 很抱歉 ! 百忙之中, 漏了資料 !" }}</div>
                </div>
            </li>
            <li v-for="(newsObj, news_i) in newsList[topicObj.key]" class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">{{ ((news_i+1) < 10 ? "0" : "") + (news_i+1) }}</div>
                <div class="list-col-grow">
                    <div class="text-black text-lg underline">{{ newsObj["topic"] }}</div>
                    <div class="text-md">{{ newsObj["content"] }}</div>
                </div>
            </li>
        </ul>
    </div>

    <!-- 郭家基金 - 使用者 -->
    <div v-if="appState === 'k_funds'" id="kuoBack" class="w-10/10 h-10/10 flex flex-col justify-center items-center bg-gray-950">
        <div id="kuoWord" class="text-9xl text-yellow-500">{{ "郭" }}</div>

        <svg width="300" height="100" viewBox="0 0 300 100">
            <path id="myPath" d="M0 0 Q150 100,300 0" fill="none" stroke="#88ce02" stroke-width="3px" />
            <circle id="myCircle1" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle2" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle3" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle4" cx="0" cy="0" r="10" fill="#88ce02" />
            <circle id="myCircle5" cx="0" cy="0" r="10" fill="#88ce02" />
        </svg>
    </div>
    
</template>

<style scoped>

</style>
