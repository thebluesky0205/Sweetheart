<template>
  <div>
    <v-card-text>
      <h1 class="text-center white--text text--accent-3">Create Account</h1>
      <v-form>
        <v-row class="pa-0 ma-0">
          <v-text-field
            dark
            label="Name"
            name="Name"
            prepend-icon="person"
            class="mr-5"
            v-model="userName"
            :rules="[v => !!v || 'Name is required']"
            type="text" />
          <v-text-field
            label="Nickname"
            name="Nickname"
            prepend-icon="mdi-account-question"
            type="text"
            v-model="userNickname"
            :rules="[v => !!v || 'Nickname is required']"
            dark />
        </v-row>
        <v-row class="pt-0 pb-0">
          <v-col class="pt-0 pb-0" cols="12" md="8">
            <v-text-field
              label="ID"
              name="ID"
              prepend-icon="mdi-account-outline"
              type="text"
              class="pa-0 ma-0"
              v-model="userId"
              :rules="[v => !!v || 'ID is required']"
              dark />
          </v-col>
          <v-col class="pt-0 pb-0" cols="12" md="2">
            <v-btn rounded color="teal" @click="idCertCheck">ID 유효성 체크</v-btn>
          </v-col>
        </v-row>
        <v-row class="pa-0 ma-0">
          <v-text-field
            label="Password"
            name="Password"
            prepend-icon="lock"
            type="password"
            class="pa-0 mr-5"
            v-model="userPassword"
            hint="비밀번호는 10글자 이상, 특수 문자를 포함해야합니다."
            :rules="passwordRules"
            dark />
          <v-text-field
            label="Password Check"
            name="Password"
            prepend-icon="lock"
            type="password"
            class="pa-0 ma-0"
            :rules="[(this.userPassword === this.userPasswordCert) || 'Password must match']"
            v-model="userPasswordCert"
            dark />
        </v-row>
        <v-row class="pa-0 ma-0">
          <v-menu
            dark
            ref="menu"
            v-model="menu"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-y
            min-width="290px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                dark
                v-model="date"
                label="Birth"
                prepend-icon="mdi-calendar"
                readonly
                v-bind="attrs"
                v-on="on"
                :rules="[v => !!v || 'Birth is required']"
                class="mr-5"
              ></v-text-field>
            </template>
            <v-date-picker
              ref="picker"
              v-model="date"
              :max="new Date().toISOString().substr(0, 10)"
              min="1950-01-01"
              @change="save"
            ></v-date-picker>
          </v-menu>
          <v-select
            :items="genders"
            label="Gender"
            name="Gender"
            dense
            dark
            prepend-icon="mdi-gender-male-female"
            class="mt-6"
            :rules="[v => !!v || 'Gender is required']"
            v-model="userGender"
          ></v-select>
        </v-row>
        <v-text-field
          label="Email"
          name="Email"
          prepend-icon="email"
          type="text"
          class="ma-0 pa-0"
          :rules="[v => !!v || 'Email is required', v => /.+@.+/.test(v) || 'E-mail must be valid']"
          v-model="userEmail"
          dark />
        <v-row>
          <v-col>
            <v-text-field
              label="우편번호"
              name="Postcode"
              prepend-icon="home"
              type="text"
              class="pa-0 ma-0"
              v-model="zonecode"
              dark />
          </v-col>
          <v-col class="justify-center">
            <v-dialog v-model="dialogPostcode" persistent max-width="600px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="primary"
                  dark
                  v-bind="attrs"
                  v-on="on"
                >
                  우편번호 찾기
                </v-btn>
              </template>
              <v-card>
                <vue-daum-postcode
                  @complete="address = $event; dialogPostcode = false; roadAddress = address.roadAddress; zonecode = address.zonecode"
                ></vue-daum-postcode>
              </v-card>
            </v-dialog>
          </v-col>
        </v-row>
        <v-text-field
          label="Address"
          name="Address"
          type="text"
          class="pa-0 ml-8 mt-0 mt-0"
          v-model="roadAddress"
          :rules="[v => !!v || 'Address is required']"
          dark />
        <v-text-field
          label="Detail Address"
          name="DetailAddress"
          type="text"
          class="pa-0 ml-8 mt-0 mt-0"
          v-model="addressDetail"
          :rules="[v => !!v || 'Detail Address is required']"
          dark />
        <v-row>
          <v-checkbox
            label="얼굴 인식에 동의하십니까?"
            color="success"
            value="success"
            class="mr-10 ml-3"
            dark
          ></v-checkbox>
          <v-dialog
            v-model="slide"
            width="500"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                color="red lighten-2"
                dark
                v-bind="attrs"
                v-on="on"
                class="mt-3"
              >
                얼굴인식
              </v-btn>
            </template>
            <v-card>
              <v-card-title class="headline grey lighten-2">
                얼굴인식
              </v-card-title>
              <v-card-text>
                얼굴인식
              </v-card-text>
              <v-divider></v-divider>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="primary"
                  text
                  @click="slide = false"
                >
                  Finish
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-row>
      </v-form>
    </v-card-text>
    <div class="text-center">
      <v-btn
        rounded
        color="teal accent-3"
        dark
        @click="signUpPrev"
      >
        SIGN UP
      </v-btn>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SecondRight',
  data () {
    return {
      slide: false,
      userName: '',
      userNickname: '',
      userId: '',
      userPassword: '',
      userPasswordCert: '',
      userBirth: '',
      userEmail: '',
      genders: ['Male', 'Female', 'Undefinded'],
      userGender: '',
      dialogPostcode: false,
      date: null,
      menu: false,
      address: null,
      zonecode: '',
      roadAddress: '',
      addressDetail: '',
      passwordRules: [
        v => !!v || 'Password is required',
        v => (v && v.length >= 8) || 'Password must have 8+ characters',
        v => /(?=.*[A-Z])/.test(v) || 'Must have one uppercase character',
        v => /(?=.*\d)/.test(v) || 'Must have one number',
        v => /([!@$%])/.test(v) || 'Must have one special character [!@#$%]'
      ]
    }
  },
  watch: {
    menu (val) {
      val && setTimeout(() => (this.$refs.picker.activePicker = 'YEAR'))
    }
  },
  props: {
    idValidate: {
      type: Boolean,
      required: true
    }
  },
  methods: {
    save (date) {
      this.$refs.menu.save(date)
    },
    idCertCheck () {
      console.log('this.userId: ' + this.userId)
      const { userId } = this
      this.$emit('idCheck', { userId })
    },
    signUpPrev () {
      if (this.userPassword !== this.userPasswordCert) {
        alert('비밀번호가 일치하지 않습니다.')
      } else {
        this.signUp()
      }
    },
    signUp () {
      if (this.idValidate === true) {
        this.totalAddress = this.roadAddress + ' ' + this.addressDetail
        console.log('total Address: ' + this.totalAddress)
        console.log('this: ' + this.userName + ', ' + this.userNickname + ', ' + this.userId + ', ' + this.date + ', ' + this.userGender + ', ' + this.userEmail + ', ' + this.totalAddress)
        const { userName, userNickname, userId, userPassword, date, userGender, userEmail, totalAddress } = this
        this.$emit('signUp', { userName, userNickname, userId, userPassword, date, userGender, userEmail, totalAddress })
      } else {
        alert('ID 중복체크를 수행하세요.')
      }
    }
  }
}
</script>
