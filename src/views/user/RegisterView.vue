<script setup lang="ts">
import { Form, type FormSubmitEvent } from "@primevue/forms";
import { zodResolver } from "@primevue/forms/resolvers/zod";
import { z } from "zod";
import { ref, watch } from "vue";
import Message from "primevue/message";
import InputText from "primevue/inputtext";
import IconField from "primevue/iconfield";
import InputIcon from "primevue/inputicon";
import VueTurnstile from "vue-turnstile";
import Card from "primevue/card";
import Button from "primevue/button";
import { postRegister, getVerifyLogin, type RegisterData } from "../../api";
import { useToast } from "primevue/usetoast";
import { useRouter } from "vue-router";
import { useRequest } from "vue-request";

const { data: loginData } = useRequest(getVerifyLogin);

watch(
    () => loginData.value,
    () => {
        if (loginData.value?.data) {
            toast.add({
                severity: "error",
                summary: "错误",
                detail: "已登录！",
                life: 3000,
            });
            setTimeout(() => {
                router.push("/");
            }, 3000);
        }
    }
);

const initialValues = ref<RegisterData>({
    username: "",
    password: "",
    confirmPassword: "",
    email: "",
    turnstileToken: "",
});

const resolver = ref(
    zodResolver(
        z
            .object({
                username: z.string().min(1, { message: "请填写此栏。" }),
                password: z
                    .string()
                    .min(1, { message: "请填写此栏。" })
                    .min(8, { message: "密码长度至少为 8 位。" }),
                confirmPassword: z.string().min(1, { message: "请填写此栏。" }),
                email: z
                    .string()
                    .min(8, { message: "请填写此栏。" })
                    .email({ message: "错误的邮箱格式。" }),
            })
            .superRefine((data, context) => {
                if (data.password !== data.confirmPassword) {
                    context.addIssue({
                        code: z.ZodIssueCode.custom,
                        path: ["confirmPassword"],
                        message: "密码不匹配。",
                    });
                }
            })
    )
);

const turnstileToken = ref("");
const turnstileRef = ref<any>();
const submitLoading = ref(false);
const toast = useToast();
const router = useRouter();
const onSubmitEvent = async (form: FormSubmitEvent) => {
    submitLoading.value = true;
    if (!form.valid) {
        submitLoading.value = false;
        return;
    }
    form.values.turnstileToken = turnstileToken.value;
    const response = await postRegister(form.values as RegisterData);
    if (response.success) {
        toast.add({
            severity: "success",
            summary: "成功",
            detail: response.message,
            life: 3000,
        });
        submitLoading.value = false;
        setTimeout(() => {
            router.push("/");
        }, 3000);
    } else {
        toast.add({
            severity: "error",
            summary: "错误",
            detail: response.message,
            life: 3000,
        });
        turnstileRef.value.reset();
        submitLoading.value = false;
    }
};
</script>

<template>
    <div class="flex flex-col items-center justify-center w-full">
        <Card class="w-[23rem] sm:w-[28rem] items-center p-4">
            <template #header>
                <h1 class="text-3xl font-bold my-8">注册</h1>
            </template>
            <template #content>
                <Form
                    v-slot="$form"
                    :resolver
                    :initialValues
                    @submit="onSubmitEvent"
                >
                    <div
                        class="flex flex-col gap-4 items-center justify-center"
                    >
                        <div class="flex flex-col gap-2">
                            <IconField>
                                <InputText
                                    class="w-[17rem] sm:w-[20rem]"
                                    name="username"
                                    autocomplete="username"
                                    type="text"
                                    placeholder="用户名"
                                >
                                </InputText>
                                <InputIcon
                                    class="icon-user-round"
                                ></InputIcon> </IconField
                            ><Message
                                v-if="$form.username?.invalid"
                                severity="error"
                                size="small"
                                variant="simple"
                                >{{ $form.username.error?.message }}</Message
                            >
                        </div>
                        <div class="flex flex-col gap-2">
                            <IconField>
                                <InputText
                                    class="w-[17rem] sm:w-[20rem]"
                                    name="email"
                                    type="text"
                                    autocomplete="email"
                                    placeholder="邮箱"
                                >
                                </InputText>
                                <InputIcon
                                    class="icon-mail"
                                ></InputIcon> </IconField
                            ><Message
                                v-if="$form.email?.invalid"
                                severity="error"
                                size="small"
                                variant="simple"
                                >{{ $form.email.error?.message }}</Message
                            >
                        </div>
                        <div class="flex flex-col gap-2">
                            <IconField>
                                <InputText
                                    class="w-[17rem] sm:w-[20rem]"
                                    name="password"
                                    autocomplete="new-password"
                                    type="password"
                                    placeholder="密码"
                                >
                                </InputText>
                                <InputIcon
                                    class="icon-key-round"
                                ></InputIcon> </IconField
                            ><Message
                                v-if="$form.password?.invalid"
                                severity="error"
                                size="small"
                                variant="simple"
                                >{{ $form.password.error?.message }}</Message
                            >
                        </div>
                        <div class="flex flex-col gap-2">
                            <IconField>
                                <InputText
                                    class="w-[17rem] sm:w-[20rem]"
                                    name="confirmPassword"
                                    autocomplete="new-password"
                                    type="password"
                                    placeholder="确认密码"
                                >
                                </InputText>
                                <InputIcon
                                    class="icon-key-round"
                                ></InputIcon> </IconField
                            ><Message
                                v-if="$form.confirmPassword?.invalid"
                                severity="error"
                                size="small"
                                variant="simple"
                                >{{
                                    $form.confirmPassword.error?.message
                                }}</Message
                            >
                        </div>
                        <div
                            class="flex flex-col w-full gap-4 items-center justify-center"
                        >
                            <p class="text-center text-sm">告诉我们你是人类</p>
                            <VueTurnstile
                                ref="turnstileRef"
                                v-model="turnstileToken"
                                site-key="0x4AAAAAAAiw3hAxhw1fzq4B"
                            ></VueTurnstile>
                        </div>
                        <Button
                            :loading="submitLoading"
                            type="submit"
                            icon="icon-user-round-plus"
                            label="注册"
                        ></Button>
                        <span class="text-center flex">
                            <p class="italic">已有账号？</p><a
                                class="text-orange-400 hover:text-orange-300 transition-colors duration-300"
                                @click="router.push('/user/login')"
                                >登录</a
                            >
                        </span>
                    </div>
                </Form>
            </template>
        </Card>
    </div>
</template>
