<script setup>
    import { ref, reactive, onMounted } from 'vue'
    
    const emit = defineEmits(['signin', 'popupMessage']);
    const props = defineProps({
        title: String,
    })

    onMounted(() => {
        console.log("Login mounted.");
        init();
    });

    let account = ref("");

    // 初始化 component
    function init(){
        console.log("Login.props.title=", props.title);
    }
    // 登入
    function signin(){
        if(account.value.toUpperCase().trim()){
            emit('signin', account.value.toUpperCase().trim()); // Emitting the event with data
        }else{
            emit('popupMessage', false, "請提供帳號~~!!"); // Emitting the event with data
        }
    }

</script>

<template>

    <div class="w-1/1 h-1/1 flex flex-col gap-2 place-content-center">
        <div class="w-1/1 h-1/3 border border-2 border-blue-300 bg-blue-900 text-white rounded-xl p-2 flex flex-col justify-center items-center">
            <span class="text-5xl underline px-10">K-Assistant</span>
            <span class="text-2xl">您的個人助理</span>
        </div>

        <div class="w-1/1 flex flex-row items-center gap-2">
            <fieldset class="fieldset flex-1">
                <legend class="fieldset-legend">What is your name?</legend>
                <input type="text" class="input w-1/1" placeholder="Type here" v-model="account" />
            </fieldset>
            <a class="flex-none border border-2 border-blue-700 bg-blue-900 rounded-xl cursor-pointer text-center text-gray-100 p-2 hover:bg-gray-200 hover:text-gray-900" @click="signin">
                <svg class="size-10" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 12H4m12 0-4 4m4-4-4-4m3-4h2a3 3 0 0 1 3 3v10a3 3 0 0 1-3 3h-2"/>
                </svg>
            </a>
        </div>
    </div>

</template>

<style scoped>

</style>
