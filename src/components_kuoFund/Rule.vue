<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String
    })

    onMounted(() => {
        console.log("Rule_KF mounted.");
        init();
    });

    let appState = ref("");
    let bankAccount = ref("123123123");

    // 初始化 component
    function init(){
        console.log("Rule_KF.init");
        console.log("Rule_KF.props.title", props.title);
        console.log("Rule_KF.props.account", props.account);
    }    
    
</script>

<template>

<div class="w-10/10 h-10/10 flex flex-col gap-2">
    <div class="w-10/10">
        <ul class="list bg-base-100 rounded-box shadow-md">
            <li class="p-4 pb-2 text-base opacity-80 tracking-wide bg-green-200">基金帳戶</li>

            <li class="list-row">
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-width="2" d="M3 21h18M4 18h16M6 10v8m4-8v8m4-8v8m4-8v8M4 9.5v-.955a1 1 0 0 1 .458-.84l7-4.52a1 1 0 0 1 1.084 0l7 4.52a1 1 0 0 1 .458.84V9.5a.5.5 0 0 1-.5.5h-15a.5.5 0 0 1-.5-.5Z"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div>Line Bank - 友感帳戶</div>
                    <div class="text-xs uppercase font-semibold opacity-60">{{ bankAccount }}</div>
                </div>
            </li>
        </ul>
    </div>

    <div class="w-10/10">
        <ul class="list bg-base-100 rounded-box shadow-md">
            <li class="p-4 pb-2 text-base opacity-80 tracking-wide bg-teal-200">繳費期限</li>
  
            <li class="list-row">
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M10 5a2 2 0 0 0-2 2v3h2.4A7.48 7.48 0 0 0 8 15.5a7.48 7.48 0 0 0 2.4 5.5H5a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h1V7a4 4 0 1 1 8 0v1.15a7.446 7.446 0 0 0-1.943.685A.999.999 0 0 1 12 8.5V7a2 2 0 0 0-2-2Z" clip-rule="evenodd"/>
                        <path fill-rule="evenodd" d="M10 15.5a5.5 5.5 0 1 1 11 0 5.5 5.5 0 0 1-11 0Zm6.5-1.5a1 1 0 1 0-2 0v1.5a1 1 0 0 0 .293.707l1 1a1 1 0 0 0 1.414-1.414l-.707-.707V14Z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <div>
                    <div>每月 10 號</div>
                </div>
                <p class="list-col-wrap text-xs">
                    表定為每月 10 號前繳交 NTD$500。( 日期可再討論 )<br />
                    每月 11 號 ~ 每月 20 號進行該月份繳款資料統整，並公告於共享 Excel、網站和 Line 群組"跟著家族去旅行"。
                </p>
            </li>
        </ul>
    </div>

    <div class="w-10/10">
        <ul class="list bg-base-100 rounded-box shadow-md">
            <li class="p-4 pb-2 text-base opacity-80 tracking-wide  bg-yellow-200">繳費方式</li>
  
            <li class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">01</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M4 3a1 1 0 0 0-1 1v8a1 1 0 0 0 1 1h1v2a1 1 0 0 0 1.707.707L9.414 13H15a1 1 0 0 0 1-1V4a1 1 0 0 0-1-1H4Z" clip-rule="evenodd"/>
                        <path fill-rule="evenodd" d="M8.023 17.215c.033-.03.066-.062.098-.094L10.243 15H15a3 3 0 0 0 3-3V8h2a1 1 0 0 1 1 1v8a1 1 0 0 1-1 1h-1v2a1 1 0 0 1-1.707.707L14.586 18H9a1 1 0 0 1-.977-.785Z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <div>
                    <div>加入"Line Bank - 友感帳戶"群組</div>
                </div>
                <p class="list-col-wrap text-xs">
                    "Remaining Reason" became an instant hit, praised for its haunting sound and emotional depth. A viral performance brought it widespread recognition, making it one of Dio Lupa’s most iconic tracks.
                </p>
            </li>
  
            <li class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">02</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M3 6a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2h-6.616l-2.88 2.592C8.537 20.461 7 19.776 7 18.477V17H5a2 2 0 0 1-2-2V6Zm4 2a1 1 0 0 0 0 2h5a1 1 0 1 0 0-2H7Zm8 0a1 1 0 1 0 0 2h2a1 1 0 1 0 0-2h-2Zm-8 3a1 1 0 1 0 0 2h2a1 1 0 1 0 0-2H7Zm5 0a1 1 0 1 0 0 2h5a1 1 0 1 0 0-2h-5Z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <div>
                    <div>以"Line 中的 iPass Money"轉帳</div>
                </div>
                <p class="list-col-wrap text-xs">
                    "Remaining Reason" became an instant hit, praised for its haunting sound and emotional depth. A viral performance brought it widespread recognition, making it one of Dio Lupa’s most iconic tracks.
                </p>
            </li>
  
            <li class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">03</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M10.915 2.345a2 2 0 0 1 2.17 0l7 4.52A2 2 0 0 1 21 8.544V9.5a1.5 1.5 0 0 1-1.5 1.5H19v6h1a1 1 0 1 1 0 2H4a1 1 0 1 1 0-2h1v-6h-.5A1.5 1.5 0 0 1 3 9.5v-.955a2 2 0 0 1 .915-1.68l7-4.52ZM17 17v-6h-2v6h2Zm-6-6h2v6h-2v-6Zm-2 6v-6H7v6h2Z" clip-rule="evenodd"/>
                        <path d="M2 21a1 1 0 0 1 1-1h18a1 1 0 1 1 0 2H3a1 1 0 0 1-1-1Z"/>
                    </svg>
                </div>
                <div>
                    <div>以常用銀行帳號轉帳</div>
                </div>
                <p class="list-col-wrap text-xs">
                    "Remaining Reason" became an instant hit, praised for its haunting sound and emotional depth. A viral performance brought it widespread recognition, making it one of Dio Lupa’s most iconic tracks.
                </p>
            </li>

            <li class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">04</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M7 6a2 2 0 0 1 2-2h11a2 2 0 0 1 2 2v7a2 2 0 0 1-2 2h-2v-4a3 3 0 0 0-3-3H7V6Z" clip-rule="evenodd"/>
                        <path fill-rule="evenodd" d="M2 11a2 2 0 0 1 2-2h11a2 2 0 0 1 2 2v7a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2v-7Zm7.5 1a2.5 2.5 0 1 0 0 5 2.5 2.5 0 0 0 0-5Z" clip-rule="evenodd"/>
                        <path d="M10.5 14.5a1 1 0 1 1-2 0 1 1 0 0 1 2 0Z"/>
                    </svg>
                </div>
                <div>
                    <div>直接給于現金代為轉入</div>
                </div>
                <p class="list-col-wrap text-xs">
                    "Remaining Reason" became an instant hit, praised for its haunting sound and emotional depth. A viral performance brought it widespread recognition, making it one of Dio Lupa’s most iconic tracks.
                </p>
            </li>
        </ul>

    </div>
</div>

</template>

<style scoped>

</style>
