<template>
  <LoadingSpinner v-if="this.$store.getters['Loading/loading']" />
  <div class="container">
    <div class="login-page">
      <div class="row">
        <div class="col-lg-6">
          <Form class="login-form" @submit="submitForm">
            <header class="login-header">
              <div class="logo-area">
                <img src="../assets/images/logo.svg" alt="로고" />
              </div>
              <h1 class="login-comment">
                나의 오프라인 이야기, OffStory와 함께 시작해 보아요!
              </h1>
              <h1 class="login-title">Login</h1>
            </header>
            <Field
              class="login-input"
              v-model="email"
              name="email"
              type="text"
              required
              rules="required|email"
              placeholder="이메일"
            />
            <div class="error-message-wrapper">
              <ErrorMessage name="email" class="error-message" />
            </div>
            <input
              class="login-input"
              v-model="password"
              name="password"
              type="password"
              placeholder="비밀번호"
              rules="required|confirmed:@password"
              required
            />
            <div class="error-message-wrapper">
              <ErrorMessage name="confirmation" class="error-message" />
            </div>
            <Button v-bind="{ width: '100%' }" class="login-button"
              >로그인</Button
            >
            <Button v-bind="{ width: '100%' }" @click="isSignup"
              >회원가입</Button
            >
          </Form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapMutations } from 'vuex'
import LoadingSpinner from '~/components/designs/LoadingSpinner'
import Button from '~/components/designs/Button'
import { loginUser, userDetailInfo, updateNameField } from '../api/index'
import { makeRandomKey } from '~/utils/function'
import { validateEmail } from '../utils/validation'
import { Field, Form, ErrorMessage } from 'vee-validate'
import { saveUserImageToCookie } from '../utils/cookies'
export default {
  data() {
    return {
      email: '',
      password: '',
      errorMessage: '',
    }
  },
  components: { Button, Field, Form, ErrorMessage, LoadingSpinner },
  computed: {
    isUserNameVaild() {
      return validateEmail(this.email)
    },
  },
  methods: {
    ...mapMutations('Loading', ['startLoading', 'endLoading']),
    async submitForm() {
      //alert('f')
      try {
        const userData = {
          email: this.email,
          password: this.password,
        }
        await this.startLoading()
        const { data } = await loginUser(userData)
        console.log('data.user', data.user)
        this.$router.push('/')
        this.$store.commit('Login/setToken', data.token)
        this.$store.commit('Login/setUsername', data.user.fullName)
        this.$store.commit('Login/setUserId', data.user._id)
        this.$storage.setItem('off_userId', data.user._id)
        this.$storage.setItem('off_auth', data.token)
        this.$storage.setItem('off_userName', data.user.fullName)

        // saveUserIdToCookie(data.user._id)
        // saveAuthToCookie(data.token)
        // saveUserToCookie(data.user.fullName)
        saveUserImageToCookie(data.user.coverImage)
        let userDetaildata = await userDetailInfo(
          this.$store.getters['Login/getUserId'],
        )

        console.log(userDetaildata)
        console.log(userDetaildata.data.coverImage)
        this.$store.commit(
          'Login/setprofileImage',
          userDetaildata.data.coverImage,
        )
        //console.log(this.$store.getters['Login/getUserProfileImage'])
        console.log(this.$store.state.Login.profileImage)

        // username을 랜덤키로 초기화
        if (!data.user.username) {
          await this.initializeUsername(data)
        }

        console.log('이미지 파일이 어떻게 저장되었을까?', data)

        // 초기화된 username 가져옴
        userDetaildata = await userDetailInfo(
          this.$store.getters['Login/getUserId'],
        )
        // 유저 데이터 storage에 저장
        this.storageSetup(userDetaildata.data)
        this.$router.go()
      } catch (error) {
        //에러 핸들링 코드
        console.log(error.response.data)
        alert('아이디 혹은 비밀번호가 맞지 않습니다.')
      }
      this.endLoading()
    },
    async initializeUsername(userData) {
      console.log('실행여부4')
      const data = {
        fullName: userData.user.fullName,
        username: makeRandomKey() + '/',
      }
      this.startLoading()
      await updateNameField(data)
      this.endLoading()
    },
    storageSetup(userData) {
      const dataToBeStored = {
        userId: userData._id,
        userCoverImage: userData.coverImage || null,
        // userIntroduction은 추후 추가 예정
        userIntroduction: userData.username,
        userFullName: userData.fullName,
        userEmail: userData.email,
        userName: userData.username,
      }
      this.$storage.setItem('userData', dataToBeStored)
    },
    isSignup() {
      this.$router.push('/signup')
    },
  },
}
</script>

<style lang="scss" scoped>
.container {
  position: relative;
  top: $LG_HEADER_HEIGHT;

  .row {
    height: calc(100vh - $LG_HEADER_HEIGHT);
    justify-content: center;

    .login {
      &-header {
        width: 100%;
        .login-comment {
          margin-top: 50px;
          font-size: $FONT_L;
          font-weight: 700;
          padding: 10px;
          text-align: center;
        }
        .login-title {
          margin-top: 50px;
          font-size: $FONT_L;
          font-weight: 700;
        }

        .logo-area {
          display: flex;
          justify-content: right;
          img {
            width: 120px;
          }
        }
        margin-bottom: $LG_PADDING_VERTICAL;
      }

      &-form {
        @include flexbox;
        flex-direction: column;
        position: relative;
        top: 50%;
        padding: $LG_PADDING_VERTICAL $LG_PADDING_HORIZONTAL;
        background-color: $COLOR_WHITE;
        box-shadow: $BOX_SHADOW;
        transform: translateY(-50%);

        .login-input {
          @include font;
          width: 100%;
          padding: $INNER_PADDING_VERTICAL $INNER_PADDING_HORIZONTAL;
          border-radius: $BORDER_RADIOUS;
          border: 1px solid $COLOR_GRAY_DARKEN;

          &::placeholder {
            font-weight: 700;
            color: $COLOR_GRAY_LIGHTEN;
          }
        }
      }

      &-button {
        margin-top: 30px;
        margin-bottom: 10px;
      }
    }
  }
}
.error-message {
  &-wrapper {
    margin-bottom: 5px;
    height: $FONT_XS;
    display: block;
    width: 100%;
    font-size: $FONT_XS;
    color: $COLOR_RED;
    z-index: 10;
  }
}
</style>
