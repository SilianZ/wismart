<script setup lang="ts">
import { RouterView, useRouter } from "vue-router";
import Button from "primevue/button";
import Toast from "primevue/toast";
import ConfirmDialog from "primevue/confirmdialog";
import { onMounted, ref } from "vue";
import { getLogout, getVerifyAdmin, getVerifyLogin } from "./api";
import { useToast } from "primevue/usetoast";
import { useRequest } from "vue-request";

const router = useRouter();
const iconClass = ref("icon-sun");
const toggleColorScheme = () => {
    let color = sessionStorage.getItem("color") == "white" ? "dark" : "white";
    sessionStorage.setItem("color", color);
    const root = document.getElementsByTagName("html")[0];
    root.classList.toggle("p-dark");
    iconClass.value = color == "white" ? "icon-sun" : "icon-moon";
};
const sha = process.env.VERCEL_GIT_COMMIT_SHA?.slice(0, 7);

const { data: loginData } = useRequest(getVerifyLogin);
const { data: adminData } = useRequest(getVerifyAdmin);
const toast = useToast();
const logoutLoading = ref(false);
const onLogoutEvent = async () => {
    logoutLoading.value = true;
    const response = await getLogout();
    if (response.success) {
        toast.add({
            severity: "success",
            summary: "成功",
            detail: response.message,
            life: 3000,
        });
        window.location.reload();
    } else {
        toast.add({
            severity: "error",
            summary: "错误",
            detail: response.message,
            life: 3000,
        });
    }
    logoutLoading.value = false;
};

onMounted(async () => {
    const color =
        sessionStorage.getItem("color") ||
        (window.matchMedia("(prefers-color-scheme: dark)").matches
            ? "dark"
            : "light");
    if (color == "dark") {
        sessionStorage.setItem("color", color);
        const root = document.getElementsByTagName("html")[0];
        root.classList.toggle("p-dark");
        iconClass.value = "icon-moon";
    }
});
</script>

<template>
    <Toast></Toast>
    <ConfirmDialog class="min-w-[17rem]">
        <template #message="slotProps">
            <div class="flex flex-col items-center w-full gap-4">
                <i :class="slotProps.message.icon" class="!text-6xl text-primary-500"></i>
                <p>{{ slotProps.message.message }}</p>
            </div>
        </template>
    </ConfirmDialog>
    <div class="sm:mx-[3rem] sm:my-[2rem] mx-[2rem] my-[1.2rem]">
        <div class="flex w-full justify-between">
            <div>
                <Button
                    icon="icon-house"
                    @click="router.push('/')"
                    severity="secondary"
                    rounded
                ></Button>
            </div>
            <div class="flex gap-2">
                <Button
                    :icon="iconClass"
                    @click="toggleColorScheme()"
                    rounded
                ></Button>
                <Button
                    v-if="adminData?.data"
                    icon="icon-shield-user"
                    label="管理员"
                    @click="router.push('/admin')"
                    rounded
                ></Button>
                <Button
                    v-if="!loginData || !loginData.data"
                    icon="icon-log-in"
                    severity="success"
                    @click="router.push('/user/login')"
                    rounded
                ></Button>
                <Button
                    v-else
                    icon="icon-log-out"
                    :loading="logoutLoading"
                    severity="danger"
                    @click="onLogoutEvent()"
                    rounded
                ></Button>
                <Button
                    v-if="!adminData?.data"
                    icon="icon-user-round-plus"
                    @click="router.push('/user/register')"
                    severity="secondary"
                    rounded
                ></Button>
            </div>
        </div>
        <div class="my-8">
            <RouterView></RouterView>
        </div>
    </div>
    <footer class="flex flex-col gap-2 items-center justify-center p-8 bg-zinc-600 text-white text-center">
        <p>
            由 MAKERs' & WisMart 共同编写。
        </p>
        <p>
           版权所有 © MAKERs' & WisMart 2025，保留所有权利。 
        </p>
        <p class="flex flex-wrap gap-1 item-center">
            构建<a
                :href="`https://github.com/SilianZ/wismart/commit/${sha}`"
                class="flex items-center gap-1 text-orange-400 hover:text-orange-300 transition-colors duration-300"
                ><i class="icon-git-commit-horizontal"></i> {{ sha }}</a
            >。
        </p>
    </footer>
</template>
