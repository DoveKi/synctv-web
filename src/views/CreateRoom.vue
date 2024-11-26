<script setup lang="ts">
import { ref } from "vue";
import { ElNotification } from "element-plus";
import router from "@/router/index";
import { createRoomApi } from "@/services/apis/room";
import { strLengthLimit } from "@/utils";
import { userStore } from "@/stores/user";

const { state: createRoomInfo, execute: reqCreateRoomApi } = createRoomApi();

const formData = ref({
  roomName: "",
  password: "",
  settings: {
    hidden: false
  }
});

const { token } = userStore();

const operateRoom = async () => {
  if (formData.value?.roomName === "") {
    ElNotification({
      title: "错误",
      message: "请填写表单完整",
      type: "error"
    });
    return;
  }
  try {
    for (const key in formData.value) {
      strLengthLimit(key, 32);
    }
    await reqCreateRoomApi({
      data: formData.value,
      headers: {
        Authorization: token.value
      }
    });
    if (!createRoomInfo.value)
      return ElNotification({
        title: "错误",
        message: "服务器并未返回token",
        type: "error"
      });
    localStorage.setItem(`room-${createRoomInfo.value.roomId}-token`, createRoomInfo.value?.token);
    if (formData.value.password)
      localStorage.setItem(`room-${createRoomInfo.value.roomId}-pwd`, formData.value.password);
    ElNotification({
      title: "创建成功",
      type: "success"
    });

    router.replace(`/cinema/${createRoomInfo.value.roomId}`);
  } catch (err: any) {
    console.error(err);
    ElNotification({
      title: "错误",
      message: err.response.data.error || err.message,
      type: "error"
    });
  }
};
</script>

<template>
  <div class="room">
    <form @submit.prevent="" class="sm:w-96 w-full">
      <input
        class="l-input"
        type="text"
        v-model="formData.roomName"
        placeholder="房间名"
        required
        autocomplete="off"
      />
      <br />
      <input
        class="l-input"
        type="password"
        v-model="formData.password"
        placeholder="房间密码"
        autocomplete="new-password"
      />
      <br />
      <button class="btn m-[10px]" @click="operateRoom()">创建房间</button>
      <div class="text-sm"><b>注意：</b>只有尊贵的DoveKi能创建房间🐱</div>
    </form>
  </div>
</template>

<style lang="less" scoped>
.room {
  text-align: center;
  margin-top: 5vmax;

  form {
    // width: 443px;
    margin: auto;

    input {
      width: 70%;

      &:hover {
        padding: 10px 15px;
        width: 74%;
      }
    }

    .btn {
      padding: 10px 15px;
      width: 70%;

      &:hover {
        padding: 12px 15px;
      }
    }
  }
}
</style>
