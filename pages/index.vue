<template>
  <v-row>
    <v-col lg="12">
      <v-card>
        <template>
          <v-data-table
            :footer-props="{ 'items-per-page-options': [20, 30, 50, 100] }"
            :page="page"
            :pageCount="numberOfPages"
            :headers="headers"
            :items="userAll"
            :loading="loading"
            loading-text="Loading... Please wait"
            class="elevation-1"
          >
            <template v-slot:top>
              <v-toolbar flat>
                <v-spacer></v-spacer>
                <v-btn
                  color="success"
                  @click="dialog = true"
                  light
                  class="mb-2"
                >
                  สร้าง
                </v-btn>
              </v-toolbar>
            </template>
            <template slot="item.manage" slot-scope="{ item }">
              <v-icon
                small
                color="red"
                class="mr-2"
                @click="deleteUser(item.userCode)"
              >
                mdi-delete
              </v-icon>
              <v-icon
                small
                color="green"
                class="mr-2"
                @click="findUser(item.userCode)"
              >
                mdi-pencil
              </v-icon>
            </template>
          </v-data-table>
        </template>
      </v-card>
    </v-col>
    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-form
        @submit.prevent="submitForm"
        ref="refForm"
        lazy-validation
        v-model="valid"
      >
        <v-card>
          <v-card-title>
            <span class="text-h5">{{ edit ? 'Update' : 'Create' }} User</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12" sm="12" md="12">
                  <v-text-field
                    id="userCode"
                    v-model="formUser.userCode"
                    label="รหัสพนักงาน"
                    outlined
                    dense
                    required
                    hide-details="auto"
                    :rules="userCodeRule"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="12" md="12">
                  <v-text-field
                    id="fullName"
                    v-model="formUser.fullName"
                    label="ชื่อ-นามสกุล"
                    outlined
                    dense
                    required
                    hide-details="auto"
                    :rules="fullNameRule"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="12" md="6">
                  <v-text-field
                    id="age"
                    v-model="formUser.age"
                    label="อายุ"
                    outlined
                    dense
                    required
                    hide-details="auto"
                    :rules="ageRule"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="12" md="6">
                  <v-select
                    v-model="formUser.sex"
                    :items="['ผู้ชาย', 'ผู้หญิง']"
                    label="เพศ"
                    outlined
                    dense
                    required
                  ></v-select>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" @click="clear" class="white--text">
              ยกเลิก
            </v-btn>
            <v-btn color="blue darken-1" type="submit" class="white--text">
              บันทึก
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-form>
    </v-dialog>
  </v-row>
</template>

<script>
export default {
  name: 'userIndex',
  data: () => ({
    dialog: false,
    headers: [
      {
        text: 'รหัสพนักงาน',
        value: 'userCode',
        sortable: true,
      },
      { text: 'ชื่อ-นามสกุล', value: 'fullName', sortable: true },
      { text: 'เพศ', value: 'sex', sortable: true },
      { text: 'อายุ(ปี)', value: 'age', sortable: true },
      { text: 'จัดการ', value: 'manage', sortable: false },
    ],
    page: 1,
    valid: true,
    edit: null,
    loading: false,
    totalPassengers: 0,
    numberOfPages: 0,
    userAll: [
      {
        userCode: 'Test-User',
        fullName: 'Test User',
        age: '12',
        sex: 'ผู้ชาย',
      },
    ],
    formUser: {
      userCode: null,
      fullName: null,
      age: null,
      sex: null,
    },
    defaultItem: {
      userCode: null,
      fullName: null,
      sex: null,
      age: null,
    },
    options: {},
    userCodeRule: [
      (v) => !!v || 'กรุณากรอกรหัสพนักงาน',
      (v) => !/[ ]/.test(v) || 'ไม่อนุญาตให้ใช้เว้นวรรค',
      (v) => /^[A-Za-z0-9\-\. \*]*$/.test(v) || 'กรุณากรอกข้อมูลให้ถูกต้อง',
    ],
    fullNameRule: [(v) => !!v || 'กรุณากรอกชื่อ-นามสกุล'],
    ageRule: [
      (v) => !!v || 'กรุณากรอกอายุ',
      (v) =>
        !v ||
        /^(\(?\+?[0-9]*\)?)?[0-9_\- \(\)]*$/.test(v) ||
        'กรุณากรอกจำนวนเต็ม',
      (v) => !v || v > 0 || 'กรุณากรอกจำนวนเต็มมากกว่า 0 ',
      (v) => !/[ ]/.test(v) || 'ไม่อนุญาตให้ใช้เว้นวรรค',
    ],
  }),
  // mounted() {
  // },
  methods: {
    async submitForm() {
      if (this.$refs.refForm.validate()) {
        if (this.edit === null) {
          const findUser = this.userAll.find(
            (e) => e.userCode === this.formUser.userCode
          )
          if (findUser === undefined) {
            this.dialog = false
            this.userAll.push({ ...this.formUser })
            this.clear()
          }
        } else {
          const findUser = this.userAll.find(
            (e) =>
              this.edit.userCode !== this.formUser.userCode &&
              e.userCode === this.formUser.userCode
          )
          if (findUser === undefined) {
            const update_user = this.userAll.findIndex(
              (e) => e.userCode === this.edit.userCode
            )
            this.userAll[update_user].userCode = this.formUser.userCode
            this.userAll[update_user].fullName = this.formUser.fullName
            this.userAll[update_user].age = this.formUser.age
            this.userAll[update_user].sex = this.formUser.sex
            this.clear()
          }
        }
      }
    },
    findUser(code) {
      if (code) {
        const findUser = this.userAll.find((e) => e.userCode === code)
        this.edit = findUser
        this.formUser = Object.assign({}, findUser)
        this.dialog = true
      }
    },
    deleteUser(code) {
      if (code) {
        const findIndex = this.userAll.findIndex((e) => e.userCode === code)
        if (findIndex > -1) {
          this.userAll.splice(findIndex, 1)
        }
      }
    },
    clear() {
      this.$nextTick(() => {
        this.formUser = Object.assign({}, this.defaultItem)
        this.edit = null
        this.dialog = false
        this.$refs.refForm.resetValidation()
      })
    },
  },
}
</script>

<style></style>
