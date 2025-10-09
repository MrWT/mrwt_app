<script setup>
    import { ref, reactive, onMounted } from 'vue'

    import SettingAward from '@/components/SettingAward.vue'
    import SettingFinance from '@/components/SettingFinance.vue'
    import SettingPersonal from '@/components/SettingPersonal.vue'
    import SettingQuiz from '@/components/SettingQuiz.vue'
    import SettingActivityKF from '@/components_kuoFund/SettingActivity.vue'
    import SettingFinanceKF from '@/components_kuoFund/SettingFinance.vue'

    const emit = defineEmits(['popupMessage']);
    const props = defineProps({
        app_state: String,
        title: String,
        account: String,
    })

    onMounted(() => {
        console.log("Setting mounted.");
        init();
    });

    let appState = ref("");
    let account = ref("");

    // 初始化 component
    function init(){
        console.log("Setting.init");
        console.log("props.app_state", props.app_state);
        console.log("props.title", props.title);
        console.log("props.account", props.account);

        appState.value = props.app_state;
        account.value = props.account;
        setTimeout(() => {
            if(document.getElementsByName("setting_tabs").length > 0){
                console.log("setting_tabs=", document.getElementsByName("setting_tabs") );
                document.getElementsByName("setting_tabs")[0].click();
            }
        }, 200);
    }
    
    // 再將 message 傳給 App.vue 
    function popupMessage(status, message){
        emit('popupMessage', status, message); // Emitting the event with data
    }
</script>

<template>

<div class="tabs tabs-border w-10/10">
    <!-- Finance -->
    <input v-if="appState === 'SET_PERSON'" type="radio" name="setting_tabs" class="tab" aria-label="設定 Finance" />
    <div v-if="appState === 'SET_PERSON'" class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <SettingFinance :account="account" @popup-message="popupMessage"></SettingFinance>
    </div>

    <!-- 個人資料 -->
    <input v-if="appState === 'SET_PERSON'" type="radio" name="setting_tabs" class="tab" aria-label="設定個人資料" />
    <div v-if="appState === 'SET_PERSON'" class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <SettingPersonal :account="account" @popup-message="popupMessage"></SettingPersonal>
    </div>

    <!-- Quiz 預設值 -->
    <input v-if="appState === 'SET_SYSTEM'" type="radio" name="setting_tabs" class="tab" aria-label="設定 Quiz 預設值" />
    <div v-if="appState === 'SET_SYSTEM'" class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <SettingQuiz @popup-message="popupMessage"></SettingQuiz>
    </div>

    <!-- Award -->
    <input v-if="appState === 'SET_SYSTEM'" type="radio" name="setting_tabs" class="tab" aria-label="設定獎項清單" />
    <div v-if="appState === 'SET_SYSTEM'" class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <SettingAward @popup-message="popupMessage"></SettingAward>
    </div>

    <!-- Finance - 郭家基金 -->
    <input v-if="appState === 'SET_SYSTEM'" type="radio" name="setting_tabs" class="tab" aria-label="設定存款/提領款項-KF" />
    <div v-if="appState === 'SET_SYSTEM'" class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <SettingFinanceKF @popup-message="popupMessage"></SettingFinanceKF>
    </div>

    <!-- 活動手冊建立 - 郭家基金 -->
    <input v-if="appState === 'SET_SYSTEM'" type="radio" name="setting_tabs" class="tab" aria-label="上傳活動手冊-KF" />
    <div v-if="appState === 'SET_SYSTEM'" class="tab-content border-base-300 bg-base-100 pt-1 px-5">
        <SettingActivityKF @popup-message="popupMessage"></SettingActivityKF>
    </div>

</div>

</template>

<style scoped>

</style>
