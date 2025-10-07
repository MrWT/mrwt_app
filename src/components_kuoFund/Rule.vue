<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import moment from 'moment'
    import { fetchData } from "@/composables/fetchData"

    const props = defineProps({
        title: String,
        account: String,
        funSetting: Object,
    })

    onMounted(() => {
        console.log("Rule_KF mounted.");
        init();
    });

    let appState = ref("");
    let bankInfo = reactive({
        bank_no: "",
        account: "",
    });
    let excelInfo = reactive({
        file_name: "",
        link: "",
    })

    // 初始化 component
    function init(){
        console.log("Rule_KF.init");
        console.log("Rule_KF.props.title", props.title);
        console.log("Rule_KF.props.account", props.account);
        console.log("Rule_KF.props.funSetting", props.funSetting);

        bankInfo.bank_no = props.funSetting.bank_info.value1;
        bankInfo.account = props.funSetting.bank_info.value2;

        excelInfo.file_name = props.funSetting.excel_info.value1;
        excelInfo.link = props.funSetting.excel_info.value2;
    }    


    /*
    key: "bank_info"
    seq: -1
    type: "kf_sys_setting"
    value1: "824",
    value2: "123123123",

    key: "excel_info"
    seq: -1
    type: "kf_sys_setting"
    value1: "XXX.xlsx",
    value2: "456456456",

    */
    
</script>

<template>

<div class="w-10/10 h-10/10 flex flex-col gap-2">
    <div class="w-10/10">
        <ul class="list bg-base-100 rounded-box shadow-md">
            <li class="p-4 pb-2 text-base opacity-80 tracking-wide bg-green-200">基金資訊</li>

            <li class="list-row">
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-width="2" d="M3 21h18M4 18h16M6 10v8m4-8v8m4-8v8m4-8v8M4 9.5v-.955a1 1 0 0 1 .458-.84l7-4.52a1 1 0 0 1 1.084 0l7 4.52a1 1 0 0 1 .458.84V9.5a.5.5 0 0 1-.5.5h-15a.5.5 0 0 1-.5-.5Z"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div>存款銀行 - Line Bank</div>
                    <div class="text-xs uppercase font-semibold opacity-60">銀行代號: {{ bankInfo.bank_no }} / 帳號: {{ bankInfo.account }}</div>
                </div>
            </li>

            <li class="list-row">
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 18h14M5 18v3h14v-3M5 18l1-9h12l1 9M16 6v3m-4-3v3m-2-6h8v3h-8V3Zm-1 9h.01v.01H9V12Zm3 0h.01v.01H12V12Zm3 0h.01v.01H15V12Zm-6 3h.01v.01H9V15Zm3 0h.01v.01H12V15Zm3 0h.01v.01H15V15Z"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div>2025年11月起, 月費: NTD$ 500</div>
                    <div class="text-xs uppercase font-semibold opacity-60">
                        依 2025/10/05(日) 中秋烤肉活動討論決議 
                    </div>
                </div>
            </li>

            <li class="list-row">
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M10 5a2 2 0 0 0-2 2v3h2.4A7.48 7.48 0 0 0 8 15.5a7.48 7.48 0 0 0 2.4 5.5H5a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h1V7a4 4 0 1 1 8 0v1.15a7.446 7.446 0 0 0-1.943.685A.999.999 0 0 1 12 8.5V7a2 2 0 0 0-2-2Z" clip-rule="evenodd"/>
                        <path fill-rule="evenodd" d="M10 15.5a5.5 5.5 0 1 1 11 0 5.5 5.5 0 0 1-11 0Zm6.5-1.5a1 1 0 1 0-2 0v1.5a1 1 0 0 0 .293.707l1 1a1 1 0 0 0 1.414-1.414l-.707-.707V14Z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div>月費繳交期限: <span class="text-red-900 font-black text-base">每月 10 號</span></div>
                    <div class="text-xs uppercase font-semibold opacity-60">
                        每戶負責人請於每月 10 號前繳交月費。( 日期可再討論 )<br />
                    </div>
                </div>
            </li>

            <li class="list-row">
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5.35709 16V5.78571c0-.43393.34822-.78571.77777-.78571H18.5793c.4296 0 .7778.35178.7778.78571V16M5.35709 16h-1c-.55229 0-1 .4477-1 1v1c0 .5523.44771 1 1 1H20.3571c.5523 0 1-.4477 1-1v-1c0-.5523-.4477-1-1-1h-1M5.35709 16H19.3571M9.35709 8l2.62501 2.5L9.35709 13m4.00001 0h2"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div>帳戶資料彙整: <span class="text-blue-900 font-black text-base">每月 20 號</span></div>
                    <div class="text-xs uppercase font-semibold opacity-60">
                        帳戶管理人於每月 20 號前進行該月份帳戶資料彙整，並公告於共享 Excel、網站和 Line 群組"跟著家族去旅行"。
                    </div>
                </div>
            </li>

            <li class="list-row">
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10V7.914a1 1 0 0 1 .293-.707l3.914-3.914A1 1 0 0 1 9.914 3H18a1 1 0 0 1 1 1v6M5 19v1a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-1M10 3v4a1 1 0 0 1-1 1H5m2.665 9H6.647A1.647 1.647 0 0 1 5 15.353v-1.706A1.647 1.647 0 0 1 6.647 12h1.018M16 12l1.443 4.773L19 12m-6.057-.152-.943-.02a1.34 1.34 0 0 0-1.359 1.22 1.32 1.32 0 0 0 1.172 1.421l.536.059a1.273 1.273 0 0 1 1.226 1.718c-.2.571-.636.754-1.337.754h-1.13"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div>帳戶細項 PDF 共享檔案</div>
                    <div class="text-xs uppercase font-semibold opacity-60">檔案名稱: {{ excelInfo.file_name }} / 共享連結: <a class="link" target="_blank" :href="excelInfo.link">{{ excelInfo.link }}</a></div>
                </div>
            </li>

        </ul>
    </div>

    <div class="w-10/10">
        <ul class="list bg-base-100 rounded-box shadow-md">
            <li class="p-4 pb-2 text-base opacity-80 tracking-wide  bg-yellow-200">月費繳交方式</li>
  
            <li class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">01</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M4 3a1 1 0 0 0-1 1v8a1 1 0 0 0 1 1h1v2a1 1 0 0 0 1.707.707L9.414 13H15a1 1 0 0 0 1-1V4a1 1 0 0 0-1-1H4Z" clip-rule="evenodd"/>
                        <path fill-rule="evenodd" d="M8.023 17.215c.033-.03.066-.062.098-.094L10.243 15H15a3 3 0 0 0 3-3V8h2a1 1 0 0 1 1 1v8a1 1 0 0 1-1 1h-1v2a1 1 0 0 1-1.707.707L14.586 18H9a1 1 0 0 1-.977-.785Z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div class="text-base">加入"Line Bank - 友感帳戶"群組</div>
                    <div class="list-col-wrap text-xs">
                        <span class="text-blue-900 font-black">* 最推薦的方式: 方便 / 透明 / 節省轉帳手續費</span><br />
                        若是本身有在使用 Line Bank 或是有興趣使用, 則可以請帳戶管理人邀請加入"友感帳戶<sup>*</sup>"群組。<br />
                        加入"友感帳戶群組"後, 可以<span class="text-blue-900 font-black">即時共管</span>基金, 
                        也方便日後移轉帳戶負責人。也可省去轉帳費用。

                        <div class="mt-5">
                            * 參考資料: 
                            <a class="link" target="_blank" href="https://corp.linebank.com.tw/blog/6659d8fb-8a09-455b-9124-494454c9772f">Line Bank - 友感帳戶</a>
                        </div>
                    </div>
                </div>
            </li>
  
            <li class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">02</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M3 6a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2h-6.616l-2.88 2.592C8.537 20.461 7 19.776 7 18.477V17H5a2 2 0 0 1-2-2V6Zm4 2a1 1 0 0 0 0 2h5a1 1 0 1 0 0-2H7Zm8 0a1 1 0 1 0 0 2h2a1 1 0 1 0 0-2h-2Zm-8 3a1 1 0 1 0 0 2h2a1 1 0 1 0 0-2H7Zm5 0a1 1 0 1 0 0 2h5a1 1 0 1 0 0-2h-5Z" clip-rule="evenodd"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div class="text-base">以"Line 中的 iPass Money"轉帳</div>
                    <div class="list-col-wrap text-xs">
                        方便直接使用 Line App 轉帳, 也可省去轉帳費用。
                    </div>
                </div>
            </li>
  
            <li class="list-row">
                <div class="text-4xl font-thin opacity-30 tabular-nums">03</div>
                <div>
                    <svg class="size-10 text-gray-800" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24">
                        <path fill-rule="evenodd" d="M10.915 2.345a2 2 0 0 1 2.17 0l7 4.52A2 2 0 0 1 21 8.544V9.5a1.5 1.5 0 0 1-1.5 1.5H19v6h1a1 1 0 1 1 0 2H4a1 1 0 1 1 0-2h1v-6h-.5A1.5 1.5 0 0 1 3 9.5v-.955a2 2 0 0 1 .915-1.68l7-4.52ZM17 17v-6h-2v6h2Zm-6-6h2v6h-2v-6Zm-2 6v-6H7v6h2Z" clip-rule="evenodd"/>
                        <path d="M2 21a1 1 0 0 1 1-1h18a1 1 0 1 1 0 2H3a1 1 0 0 1-1-1Z"/>
                    </svg>
                </div>
                <div class="list-col-grow">
                    <div class="text-base">以常用銀行帳號轉帳</div>
                    <div class="list-col-wrap text-xs">
                        可能需自行負擔轉帳手續費。
                    </div>
                </div>
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
                <div class="list-col-grow">
                    <div class="text-base">直接給予現金代為轉入</div>
                    <div class="list-col-wrap text-xs">
                        <span class="text-red-900 font-black">* 最不推薦的方式</span>
                    </div>
                </div>
            </li>
        </ul>

    </div>
</div>

</template>

<style scoped>

</style>
