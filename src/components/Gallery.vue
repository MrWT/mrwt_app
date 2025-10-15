<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import { gsap } from "gsap"
    import { DrawSVGPlugin } from "gsap/DrawSVGPlugin"
    import { MotionPathPlugin } from 'gsap/MotionPathPlugin'
    import { getRandomNumber } from "@/composables/random"

    const props = defineProps({
        title: String,
        account: String,
        cname: String,
        user_role: String,
    })

    onMounted(() => {
        console.log("gallery.mounted");
        init();

        setTimeout(() => {
            window.dispatchEvent(new Event('resize'));

        }, 1000);

    });

    let appState = ref("");
    // 選擇的圖片 url
    let selImgUrl = ref("");
    // 生成的 channels
    let channelCount = 1;
    let channels = reactive([]);
    // 圖片候選清單
    let imageCount = 100;
    let candidateImageUrls = [
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-1.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-2.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-3.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-4.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-5.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-6.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-7.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-8.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-9.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-10.jpg",
        "https://flowbite.s3.amazonaws.com/docs/gallery/masonry/image-11.jpg",
    ];
    let imageUrls = [];

    let account = ref("");
    let cname = ref("");
    let user_role = ref("");
    let repeatRows = ref(1000);
    let repeatWords_3 = ref(3);
    let repeatWords_4 = ref(4);
    let repeatWords_6 = ref(6);
    let repeatWords_7 = ref(7);

    // 初始化 component
    function init(){
        console.log("Gallery.init");
        console.log("Gallery.props.title=", props.title);
        console.log("Gallery.props.account=", props.account);
        console.log("Gallery.props.cname=", props.cname);
        console.log("Gallery.props.user_role=", props.user_role);
        account.value = props.account;
        cname.value = "郭"; // props.cname;
        user_role.value = props.user_role;

        if(account.value !== "KUOFAMILY" && user_role.value !== "admin_kf"){
            // 依據 imageCount 生成 imageUrls
            imageCount = getRandomNumber(candidateImageUrls.length, 100);
            for(let img_i = 0; img_i < imageCount; img_i++){
                let imgIndex = getRandomNumber(0, candidateImageUrls.length -1);
                let imgUrl = candidateImageUrls[ imgIndex ];
                imageUrls.push( imgUrl );
            }

            // w-sm 約等於 384px
            // w-md 約等於 448px
            //console.log("window.innerWidth=" + window.innerWidth);
            if(window.innerWidth > 448){
                channelCount = getRandomNumber(3, 4);
            }else if(304 < window.innerWidth && window.innerWidth < 448 ){
                channelCount = getRandomNumber(1, 2);
            }else{
                channelCount = getRandomNumber(1, 1);
            }
            // 依據 channelCount 生成 channel
            for(let c_i = 0; c_i < channelCount; c_i++){
                channels.push( [] );
            }
            // 建立 channel 內容
            genChannels();
        }else{

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
    }
    // 建立 channel
    function genChannels(){
        // 清空 channels
        for(let c_i = 0; c_i < channelCount; c_i++){
            channels[c_i].splice(0, channels[c_i].length);
        }

        let imgCount = imageUrls.length / channelCount;
        console.log("imgCount=", imgCount);

        let channelIndex = 0;
        let imgGrpIndex = 0;
        imageUrls.forEach((imgSrc, is_i) => {
            imgGrpIndex = imgGrpIndex % imgCount;
            channelIndex += imgGrpIndex === 0 ? 1 : 0;
            channelIndex = channelIndex % channelCount;

            channels[channelIndex].push( imgSrc );
        });
    }
    // showModal
    function showModal(imgUrl){
        selImgUrl.value = imgUrl;
        document.getElementById("imgModal").showModal();
    }

</script>

<template>
    <!-- 一般使用者 -->
    <div v-if="account !== 'KUOFAMILY' && user_role !== 'admin_kf'" class="grid gap-4"
    :class="{ 'grid-cols-1': channels.length === 1,
            'grid-cols-2': channels.length === 2,
            'grid-cols-3': channels.length === 3,
            'grid-cols-4': channels.length === 4 }">
        <div v-for="(channel, c_i) in channels" class="grid"
        :class="{ 'gap-16': channels.length === 1,
                    'gap-20': channels.length === 2,
                    'gap-20': channels.length === 3,
                    'gap-20': channels.length === 4 }">
            <div v-for="(imgSrc, is_i) in channel">
                <img class="h-auto max-w-full rounded-lg shadow-2xl" :src="imgSrc" alt="" @click="showModal(imgSrc)">
            </div>
        </div>
    </div>

    <dialog id="imgModal" class="modal">
        <div class="modal-box w-10/10 h-8/10">
            <h3 class="text-lg font-bold">Hello!</h3>
            <div class="w-10/10 h-8/10 justify-items-center content-center">
            <img :src="selImgUrl" class="max-h-full object-fill rounded-lg" />
            </div>

            <div class="modal-action">
            <form method="dialog">
                <!-- if there is a button in form, it will close the modal -->
                <button class="btn">Close</button>
            </form>
            </div>
        </div>
    </dialog>

    <!-- 郭家基金 - 使用者 -->
    <div v-if="account === 'KUOFAMILY' || user_role === 'admin_kf'" id="kuoBack" class="w-10/10 h-10/10 flex flex-col justify-center items-center bg-gray-950">
        <div id="kuoWord" class="text-9xl text-yellow-500">{{ cname }}</div>

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
