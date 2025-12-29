<script setup>
    import { ref, reactive, onMounted } from 'vue'
    
    const emit = defineEmits(['signin']);
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
        emit('signin', account.value.toUpperCase().trim()); // Emitting the event with data
    }

</script>

<template>

    <div class="w-1/1 h-1/1 flex flex-col gap-2 place-content-center">
        <div class="w-1/1 h-1/3 border border-2 border-blue-300 bg-blue-900 text-white rounded-xl p-2 text-3xl place-content-center">
            歡迎光臨~~!<br />
            K-Assistant 是您的個人資料管理助理 <br />
        </div>

        <div class="w-1/1 flex flex-row items-center gap-2">
            <span class="flex-none text-xl">帳號:</span>
            <input type="text" class="flex-1 border rounded-xl p-2" v-model="account" />
            <a class="flex-none border border-2 border-blue-700 bg-blue-500 rounded-xl cursor-pointer text-center text-gray-100 p-2 hover:bg-gray-200 hover:text-gray-900" @click="signin">
                <span class="text-xl">登入</span>
            </a>
        </div>
    </div>

</template>

<style scoped>

</style>
