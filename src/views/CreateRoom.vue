<script setup lang="ts">
import { ref } from "vue";
import { ElNotification } from "element-plus";
import router from "@/router/index";
import { createRoomApi } from "@/services/apis/room";
import { strLengthLimit } from "@/utils";
import { userStore } from "@/stores/user";
import { RoomStatus } from "@/types/Room";

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
        message: "服务器并未返回房间信息",
        type: "error"
      });

    switch (createRoomInfo.value.status) {
      case RoomStatus.Active:
        if (formData.value.password)
          localStorage.setItem(`room-${createRoomInfo.value.roomId}-pwd`, formData.value.password);
        ElNotification({
          title: "创建成功",
          type: "success"
        });
        router.replace(`/cinema/${createRoomInfo.value.roomId}`);
        break;
      case RoomStatus.Pending:
        ElNotification({
          title: "创建成功",
          message: "房间正在审核中，请等待管理员审核",
          type: "warning"
        });
        break;
      case RoomStatus.Banned:
        ElNotification({
          title: "创建失败",
          message: "房间被封禁",
          type: "error"
        });
        break;
      default:
        ElNotification({
          title: "创建失败",
          message: "未知错误",
          type: "error"
        });
    }
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
      <div>
        <input class="w-auto" type="checkbox" v-model="formData.settings.hidden" />
        <label title="不显示在房间列表">&nbsp;是否隐藏此房间</label>
      </div>
      <button class="btn m-[10px]" @click="operateRoom()">创建房间</button>
      <div class="text-sm"><b>注意：</b>所有输入框最大只可输入32个字符</div>
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
