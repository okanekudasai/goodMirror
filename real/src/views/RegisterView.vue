<template>
  <div>
    <div id="main_logo">
      <img src="@/assets/main_logo2.png" alt="">
    </div>
    
    <div class="regiterview">
      <div id="container">
        <div class="header_menu" style="font-size: 25px;">
          프로필 입력
        </div>
        <div class="header_menu" id="profile_image">
          <div v-if="this.img_url">
            <div id="profile_background">
              <img id="preview_img" :src="this.img_url" />
            </div>
            <div>
              <button @click="clearImage">다시 선택</button>
            </div>
          </div>
          <div v-else>
            <img id="blank_img" src="@/assets/profile_default_with_cam.svg" alt="" @click="profile_click">
          </div>
          <input ref="file_input" type="file" @change="oneFileSelect" style="display:none" />
        </div>
        <div id="input_box" style="display: flex; align-items: center;">
          <div style="margin-right: 19px;">닉네임</div>
          <div>
            <input id="nickname_input" ref="nickname_input" v-model="nickname" maxlength="6">
          </div>
        </div>
        <div v-if="this.nickname.length">
          <div v-if="!can_use" style="color: red;">이미 사용 중입니다</div>
          <div v-else style="color: green;">사용 할 수 있습니다</div>
        </div>
        <div id="input_box" style="display: flex; align-items: center;">
          <div style="margin-right: 5px;">전화번호</div>
          <div>
            <input id="nickname_input" ref="nickname_input" v-model="phone" maxlength="12">
          </div>
        </div>
        <div class="notice grey">전화번호는 '-' 없이 숫자만 입력해주세요.</div>
        <div>
          <button id="save_button" @click="save">저장</button>
        </div>
      </div>
    </div>
</div>
</template>

<script>

export default {
  name: 'RegisterView',
  data() {
    return {
      email: '',
      nickname: '',
      phone: '',
      can_use: false,
      one_file: undefined,
      img_url: undefined,
    }
  },
  methods: {
    check() {
      // console.log(this.nickname);
      this.axios.get(process.env.VUE_APP_SPRING + "user/nickname/" + this.nickname)
        .then(({ data }) => {
          if (data.code) {
            this.can_use = false;
          } else {
            this.can_use = true;
          }
        })
    },
    profile_click() {
      this.$refs.file_input.click();
    },
    oneFileSelect(e) {
      this.one_file = e.target.files[0];
      this.img_url = URL.createObjectURL(e.target.files[0]);
      // console.log("files : " + e.target.files[0]);
      // console.log("one_file : " + this.one_file.path);
      // console.log("e : " + e);
      // console.log("img_url : " + this.img_url);
    },
    save() {
      if (this.nickname.length == 0) {
        alert("닉네임을 확인해 주세요");
        return;
      } else if (this.can_use == false) {
        alert("이미 사용 중인 닉네임이에요");
        return;
      }
      const formData = new FormData();
      formData.append("nickname", this.nickname);
      formData.append("email", this.email);
      formData.append("phone", this.phone);

      if (this.one_file) {
        formData.append("file", this.one_file);
      }

      // console.log("phone : ", this.phone);
      // console.log("file : " + this.nickname);
      // console.log("this.file : " + this.img_url);
      // console.log("this.file : " + this.one_file.get());
      

      this.axios.post(process.env.VUE_APP_SPRING + "kakao/register", formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        }
      })
        .then((res) => {
          // const code = res.data.code
          // if(code) {
          sessionStorage.setItem('token', res.data)
          // console.log('회원가입 성공')
          // console.log(res)
          // alert("등록완료");
          this.$router.push({ name: "home" });
          // } else {
          //   console.log(res)
          //   console.log('code err')
          // }
        })
        .catch((e) => {
          // console.log('회원가입 실패')
          console.log(e)
        })
    },

    reSelect() {
      alert("다시선택");
    },
    clearImage() {
      this.$refs.file_input.value = ''
      this.one_file = undefined;
      this.img_url = undefined;
    }
  },

  watch: {
    'nickname': 'check'
  },
  created() {
    // console.log(this.$route.params.email);
    this.email = this.$route.params.email;
  }
}
</script>

<style lang="scss" scoped>
$input_height: 40px;
$button_height: 40px;
$logo_size: 120px;
$image_size: 140px;
$div_interval: 40px;


// position: absolute;
//   top: 2%;
//   right: 10%;
//   border: solid 1px black;
//   padding: 1.5% 3%;

#main_logo {
  position: absolute;
  top: 2%;
  left: 50%;
  transform: translateX(-50%);
}

#main_logo>img {
  width: $logo_size;
}

#blank_img {
  width: $image_size;
}

#container {
  transform: translateY(-5%);
  margin-top:20%
}

#container div {
  text-align: center;
}

#input_box {
  margin-top: $div_interval;
}

#nickname_input {
  background-color: rgba(67, 64, 57, 0.3);
  height: $input_height;
  width: calc(100vw * 0.5);
  border-radius: $input_height / 2;
  color: #1D182C;
}

#save_button {
  margin-top: $div_interval;
  background-color: #1D182C;
  color: white;
  height: $button_height;
  padding: 0 20px;
  border-radius: $button_height / 2;
  font-size: 20px;
}

#profile_background {
  // background-color: rgb(24, 138, 28);
  margin: 20px auto;
  height: $image_size * 0.8;
  width: $image_size * 0.8;
  border-radius: $image_size / 2;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

#preview_img {
  width: 100%;
  height: 100%;
}

.notice {
  margin: 20px;
}

.grey {
  color: gray;
}

.regiterview {
  height: 80vh;
  margin-top: 200px;
  width: 100%;
}


</style>