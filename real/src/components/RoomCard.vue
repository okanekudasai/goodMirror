<template>
  <div id="container">
    <div class="wooden-sign">
      <img src="@/assets/wooden_sign.png" />
    </div>
    <div class="room-content" @click="enterRoom">
      <div style="display: flex; align-items: center">
        <img src="@/assets/Lock.svg" v-if="room.password" />
        <img src="@/assets/Unlock.svg" v-else />
        <div style="margin-left: 10px" class="room-title">
          {{ room.title }}({{ room_info.playercnt }})
        </div>
      </div>
      <div style="margin-top: 10px"></div>
      <!-- stretch는 위아래로 모든 div의 높이를 같게 해줘요 -->
      <div style="display: flex; align-items: stretch">
        <!-- flex: 1 은 남은공간을 이 div가 모두 가져가게 해 줘요 -->
        <div class="card_menu flex_center" style="color: white; flex: 1">
          <img src="@/assets/Clock.svg" />
          <!-- <div class="start_time" v-if="this.room.room_start_time < Date.now()">진행중</div> -->
          <div class="start_time">
            {{
              `${this.dateTime.month}월 ${this.dateTime.day}일 ${this.dateTime.hour}시 ${this.dateTime.minute}분`
            }}
          </div>
        </div>

        <div class="card_margin"></div>

        <div v-if="!isPushed" class="card_menu" @click="bell_icon">
          <img src="@/assets/Notification.svg" />
        </div>
        <div v-if="isPushed" class="card_menu" @click="bell_icon">
          <img src="@/assets/Notification_ringing.svg" />
        </div>

        <div class="card_margin"></div>

        <div class="card_menu" @click="info_icon">
          <img src="@/assets/Info.svg" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    room_info: Object,
    isEnter: Boolean,
  },

  data() {
    return {
      room: {},
      isPushed: false,
      dateTime: {
        month: 0,
        day: 0,
        hour: 0,
        minute: 0,
      },
    };
  },
  async created() {
    // console.log("나 시작해요")
    this.room = await this.axios
      .get(process.env.VUE_APP_SPRING + "room/" + this.room_info.roomid, {
        headers: { Authorization: sessionStorage.getItem("token") },
      })
      .then((res) => res.data.result);
        // console.log("나 끝했어요")
    this.dateTime = this.room.dateTime;
    this.isPushed = this.room.isPushed;
    // console.log("room", this.room);
    // console.log("room_info", this.room_info);
    // console.log("this.room_info.isPushed", this.room.isPushed);
  },
  methods: {
    enterRoom() {
      // console.log("room", this.room);
      // console.log("this.room.password", this.room.password);
      if (!this.room.password) {
        this.goWaitingRoom();
      } else {
        this.$emit("room_info", this.room);
        this.$emit("show_pass");
      }
    },
    async goWaitingRoom() {
      var user = await this.axios
        .get(process.env.VUE_APP_SPRING + "user", {
          headers: { Authorization: sessionStorage.getItem("token") },
        })
        .then((res) => res.data.result);
      // console.log("-------------", user);
      this.sendData({
          event: "enter",
          room: this.room_info.roomid,
          nickname: user.nickname,
          level: user.level,
        }
      );
      this.$router.push({ path: "/waitingRoom/" + this.room.id });
    },
    bell_icon(e) {
      if (!this.isPushed) {
        this.axios
          .post(
            process.env.VUE_APP_SPRING + "room/push/" + this.room_info.roomid,
            "",
            {
              headers: { Authorization: sessionStorage.getItem("token") },
            }
          )
          .then(({ data }) => {
            if (data.code == 1) {
              this.isPushed = true;
              // console.log("예약 성공");
            } else {
              alert("예약에 실패했습니다. 오류코드 : " + data.code);
            }
          });
      } else {
        this.axios
          .delete(
            process.env.VUE_APP_SPRING + "room/push/" + this.room_info.roomid,
            {
              headers: { Authorization: sessionStorage.getItem("token") },
            }
          )
          .then(({ data }) => {
            if (data.code == 1) {
              this.isPushed = false;
              // console.log("예약취소 성공");
            } else {
              alert("예약 취소에 실패했습니다. 오류코드 : " + data.code);
            }
          });
      }
      e.stopPropagation();
    },
    info_icon(e) {
      // console.log(this.room);
      this.$emit("info_show");
      this.$emit("room_info", this.room);
      e.stopPropagation();
    },
  },
};
</script>

<style lang="scss" scoped>
@import "@/assets/scss/_variable.scss";

#container {
  margin: $card_vertical_margin 0;
  // background: #f2f2f2;
  // background-image: url('@/assets/wooden_sign.png');
  // background-size: cover;
  height: 120px;
  width: 397px;
  border-radius: 15px;
  padding: 20px 60px;
  box-shadow: 0px 10px 20px -10px rgb(0, 0, 0, 0.7);
  animation-name: room_card_appear;
  animation-duration: 0.5s;
  animation-iteration-count: 1;
  position: relative;
}
.room-title {
  color: rgb(255, 251, 226);
  text-shadow: 1px 1px 0px rgb(49, 49, 49), -1px -1px 0px rgb(57, 57, 57);
  font-size: 1.4rem;
}
.room-content {
  position: relative;
  cursor: pointer;
  z-index: 1;
}
.wooden-sign {
  position: absolute;
  z-index: 0;
  top: -75px;
  width: 400px;
  height: 234px;
  left: 0px;
}
.wooden-sign img {
  width: 100%;
  height: 100%;
}
@keyframes room_card_appear {
  0% {
    opacity: 0;
    transform: translateY(-50px);
  }
}
.card_menu {
  padding: 10px;
  border-radius: 5px;
  background-color: #291d00e8;
  text-shadow: 1px 1px 0 #6a5c00, -1px -1px 0 #4e4300;
  box-shadow: 1px 1px 0 #746000, -1px -1px 0 rgb(204 181 96 / 78%);
}
.card_menu img {
  width: 22px;
  height: 22px;
}
.card_margin {
  margin-left: $card_margin;
}
.start_time {
  width: 130px;
  text-align: center;
}
</style>