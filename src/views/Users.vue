<template>
  <div class="wrapper">
    <div class="list">
      <span>Фильтры</span>
      <b-form-input class="searchName" v-model="searchUser" placeholder="Имя кандидата"></b-form-input>
      <div style="position: relative">
        <div v-if="!filterJob" class="filterLabel">Вакансия</div>
        <b-form-select
          class="form-control"
          style="width: 84%; margin: 8px"
          v-model="filterJob"
          :options="jobList"
          value-field="item"
          text-field="name"
          disabled-field="notEnabled"
        ></b-form-select>
        <b-icon @click="() => filterJob = 0" icon="x-square-fill" variant="error" style="fill: red; right: 20px; position: absolute; margin-top: -34px;"></b-icon>
      </div>
      <div style="position: relative">
        <div v-if="!filterStatus" class="filterLabel">Статус</div>
        <b-form-select
          class="form-control"
          style="width: 84%; margin: 8px"
          v-model="filterStatus"
          :options="statusOptions"
          value-field="item"
          text-field="name"
          disabled-field="notEnabled"
        ></b-form-select>
        <b-icon @click="() => filterStatus = 0" icon="x-square-fill" variant="error" style="fill: red; right: 20px; position: absolute; margin-top: -34px;"></b-icon>
      </div>
      <hr />
      <!--      -->
      <div
        @click="() => setSelectedId(item.id)"
        v-for="item in filteredList"
        :key="item.id"
        class="item"
      >
        <b-avatar variant="info" :src="item.avatar"></b-avatar>
        <div>{{item.name}}</div>
        <div v-if="item.status === '8'" class="success" :class="{'isDelUser': isActive(item.vacation)}"></div>
        <div v-else-if="item.status === '9'" class="fail" :class="{'isDelUser': isActive(item.vacation)}"></div>
        <div v-else class="progress" :class="{'isDelUser': isActive(item.vacation)}"></div>
      </div>
    </div>
    <div class="data">
      <div v-if="selectedId">
        <div class="buttons">
          <b-button size="sm" @click="downloadResume" variant="outline-primary">
            <b-icon icon="file-earmark-arrow-down" aria-hidden="true"></b-icon> Резюме
          </b-button>
          <b-button :disabled="isActive(selectedData.vacation)" size="sm" v-b-modal.modal-3 variant="outline-primary">
            <b-icon icon="person-circle" aria-hidden="true"></b-icon> Статус
          </b-button>
          <b-button size="sm"  v-b-modal.modal-5 variant="outline-primary">
            <b-icon icon="question-circle" aria-hidden="true"></b-icon> Тесты
          </b-button>
          <b-button size="sm"  v-b-modal.modal-2 variant="outline-primary">
            <b-icon icon="envelope" aria-hidden="true"></b-icon> Написать
          </b-button>
          <b-button size="sm"  v-b-modal.modal-4 variant="outline-primary">
            <b-icon icon="chat-right-text" aria-hidden="true"></b-icon> Комментарий
          </b-button>
          <b-button v-b-modal.modal-1 @click="editUserId = selectedData.id" variant="primary">
            <b-icon icon="pencil-square" aria-hidden="true"></b-icon> Изменить
          </b-button>
          <b-button variant="danger" @click="() => deleteData(selectedData.id)">
            <b-icon icon="trash" aria-hidden="true"></b-icon>
          </b-button>
        </div>
        <div class="mainInfo">
          <div class="photo">
            <img :src="selectedData.avatar">
          </div>
          <div style="text-align: left">
            <div class="name">{{selectedData.name}}</div>
            <div>
              <span class="userLabel">Возраст:</span>
              <span class="age">{{selectedData.age}}</span>
            </div>
            <div>
              <span class="userLabel">Город:</span>
              <span class="age">{{selectedData.city}}</span>
            </div>
            <div>
              <span class="userLabel">Пол:</span>
              <span class="age">{{selectedData.male}}</span>
            </div>
            <div>
              <span class="userLabel">Почта:</span>
              <span class="age">{{selectedData.email}}</span>
            </div>
            <div>
              <span class="userLabel">Телефон:</span>
              <span class="age">{{selectedData.phone}}</span>
            </div>
            <div>
              <span class="userLabel">Образование:</span>
              <span class="age">{{selectedData.education}}</span>
            </div>
            <div>
              <span class="userLabel">Статус:</span>
              <span class="age">{{selectedStatus}}</span>
            </div>
            <div>
              <span class="userLabel">Рекрутер:</span>
              <span class="age">{{selectedData.hrEmail}}</span>
            </div>
            <div>
              <span class="userLabel">Вакансия:</span>
              <span class="age">{{selectedData.vacation}}</span>
            </div>
          </div>
        </div>
        <div class="user-about">
          {{selectedData.about}}
        </div>
      </div>
      <div v-else>
        <img src="../assets/empty.jpg">
      </div>
    </div>
    <div class="add">
      <b-button v-b-modal.modal-1 @click="clearData" variant="success">
        <b-icon icon="plus-lg"></b-icon>
      </b-button>
    </div>
    <b-modal id="modal-1" title="BootstrapVue">
      <template #modal-header>
        <div class="head"><h5>Создать нового кандидата</h5></div>
      </template>

      <template #default>
        <div class="content">
          <div style="display: flex; gap: 16px">
            <FileInput :value="newUser.avatar" @change="saveAvatar"/>
            <FileInputTwo @change="saveResume"/>
          </div>
          <div class="row">
            <b-form-input style="width: 355px" v-model="newUser.name" placeholder="ФИО кандидата"></b-form-input>
            <div style="position: relative; width: 100px;padding: 0">
              <div v-if="!newUser.male" class="filterLabel">Пол</div>
              <b-form-select
                class="form-control"
                v-model="newUser.male"
                :options="optionsMale"
                value-field="item"
                text-field="name"
                disabled-field="notEnabled"
              ></b-form-select>
            </div>
          </div>
          <div class="row">
            <b-form-input v-model="newUser.age"  type="number" placeholder="Возраст"></b-form-input>
            <b-form-input v-model="newUser.city" placeholder="Город проживания"></b-form-input>
          </div>
          <div class="row">
            <b-form-input v-model="newUser.email" type="email" placeholder="Почта"></b-form-input>
            <b-form-input v-model="newUser.phone" type="tel" placeholder="Номер телефона"></b-form-input>
          </div>
          <b-form-input v-model="newUser.education" placeholder="Образование"></b-form-input>
          <b-form-input v-model="newUser.about" placeholder="О себе"></b-form-input>
          <div class="label">Вакансия, на которую претендует кандидат</div>
          <div style="position: relative">
            <div v-if="!newUser.vacation" class="filterLabel">Вакансия</div>
            <b-form-select
              class="form-control"
              v-model="newUser.vacation"
              :options="jobList"
              value-field="item"
              text-field="name"
              disabled-field="notEnabled"
            ></b-form-select>
          </div>
        </div>
      </template>

      <template #modal-footer="{ ok, cancel }" class="footer">
        <b-button
          variant="success"
          :disabled="!newUser.name || !newUser.email || !newUser.phone || !newUser.avatar || !newUser.male || !newUser.vacation"
          @click="() => saveData(ok)"
        >
          Сохранить
        </b-button>
        <b-button @click="cancel()">
          Отменить
        </b-button>
      </template>
    </b-modal>
    <b-modal id="modal-2" title="BootstrapVue">
      <template #modal-header>
        <div class="head"><h5>Отправить письмо на почту</h5></div>
      </template>

      <template #default>
        <div>
          <b-form-textarea
            v-model="mailText"
            placeholder="Введите сообщение"
            rows="3"
            max-rows="6"
          ></b-form-textarea>
          <span style="color: #858585">Сообщение может попасть в спам!</span>
        </div>
      </template>

      <template #modal-footer="{ ok, cancel }" class="footer">
        <b-button variant="success" @click="() => {sendMailTo(); ok()}">
          Отправить
        </b-button>
        <b-button @click="cancel()">
          Отменить
        </b-button>
      </template>
    </b-modal>
    <b-modal id="modal-3" title="BootstrapVue">
      <template #modal-header>
        <div class="head"><h5>Статус кандидата</h5></div>
      </template>

      <template #default>
        <div style="display:flex; flex-direction: column; align-items: center;justify-content: center">
          <b-form-select
            class="form-control"
            style="width: 250px"
            v-model="selectedData.status"
            :options="statusOptions"
            @change="setStatus"
            value-field="item"
            text-field="name"
            disabled-field="notEnabled"
          ></b-form-select>
          <span style="color: #858585">Для обновления статуса нажмите на поле ввода</span>
        </div>
      </template>

      <template #modal-footer="{ ok, cancel }" class="footer">
        <b-button variant="success" @click="ok()">
          Сохранить
        </b-button>
        <b-button @click="cancel()">
          Отменить
        </b-button>
      </template>
    </b-modal>
    <b-modal id="modal-4" title="BootstrapVue">
      <template #modal-header>
        <div class="head"><h5>Комментарий</h5></div>
      </template>

      <template #default>
        <div>
          <div v-for="item in commentList" :key="item.hr">
            <span style="font-weight: bold">{{item.hr}}</span>: <i>{{item.text}}</i>
          </div>
          <b-form-textarea
            v-model="comment"
            placeholder="Введите комментарий"
            rows="3"
            max-rows="5"
          ></b-form-textarea>
        </div>
      </template>

      <template #modal-footer="{ ok, cancel }" class="footer">
        <b-button variant="success" @click="saveComment(); ok()">
          Сохранить
        </b-button>
        <b-button @click="cancel()">
          Отменить
        </b-button>
      </template>
    </b-modal>
    <b-modal id="modal-5" title="BootstrapVue">
      <template #modal-header>
        <div class="head"><h5>Решенные тесты кандидата</h5></div>
      </template>

      <template #default>
        <div>
          <div style="display:flex; justify-content: center">
            <FileInputTwo @change="saveTest"/>
          </div>
          <div v-if="selectedData.test">
            <b-button size="sm" @click="downloadTest" variant="outline-primary">
              <b-icon icon="file-earmark-arrow-down" aria-hidden="true"></b-icon> Скачать
            </b-button>
          </div>
        </div>
      </template>

      <template #modal-footer="{ ok, cancel }" class="footer">
        <b-button variant="success" @click="updateData(); ok()">
          Сохранить
        </b-button>
        <b-button @click="cancel()">
          Отменить
        </b-button>
      </template>
    </b-modal>
  </div>
</template>

<script>
import download from 'downloadjs'
import { getDatabase, ref, child, get, set, update } from 'firebase/database'
import FileInput from '@/components/FileInput'
import FileInputTwo from '@/components/FileInput-2'
import { sendEmail, sendUpdatedStatus } from '@/helpers/sendEmail'
import { logger } from '@/helpers/logger'
export default {
  mounted () {
    if (!this.$store.getters.getEmail) {
      this.$router.push('/')
    }
    this.loadData()
  },
  components: {
    FileInput,
    FileInputTwo
  },
  watch: {
    editUserId () {
      if (!this.editUserId) {
        return
      }
      this.newUser = { ...this.selectedData }
    }
  },
  computed: {
    filteredList () {
      const filterName = this.searchUser
        ? this.list.filter((item) => item.name.includes(this.searchUser))
        : this.list
      const filterJob = this.filterJob
        ? filterName.filter((item) => item.vacation === this.filterJob)
        : filterName
      const filteredStatus = this.filterStatus
        ? filterJob.filter((item) => +item.status === +this.filterStatus)
        : filterJob
      return filteredStatus
    },
    commentList () {
      const hrnow = this.$store.getters.getEmail.split('@')[0]
      const list = Object.entries(this.selectedData.comment).map(([key, value]) => ({
        hr: key,
        text: value
      })).filter((item) => item.hr !== hrnow)
      return list
    },
    selectedData () {
      return this.list.find((item) => item.id.toString() === this.selectedId.toString())
    },
    selectedStatus () {
      const test = this.statusOptions.find((item) => +item.item === +this.selectedData.status)
      return test && test.name ? test.name : this.selectedData.status
    }
  },
  methods: {
    setSelectedId (id) {
      this.selectedId = id.toString()
      const hrnow = this.$store.getters.getEmail.split('@')[0]
      this.comment = this.selectedData.comment[hrnow] || ''
    },
    isActive (vacantName) {
      const vacant = this.jobList.find((item) => item.name === vacantName)
      if (!vacant) {
        return false
      }
      return !!vacant.isDel || false
    },
    downloadResume () {
      const name = this.selectedData.name.toString().replace(' ', '_') + '.doc'
      download(this.selectedData.resume, name)
    },
    downloadTest () {
      const name = this.selectedData.name.toString().replace(' ', '_').slice(0, 5) + '_test' + '.doc'
      download(this.selectedData.test, name)
    },
    sendMailTo () {
      sendEmail(this.mailText, this.selectedData.email)
      logger(this.$store.getters.getEmail, 'Письмо на почту', this.mailText)
    },
    setStatus () {
      sendUpdatedStatus(this.selectedData.status, this.selectedData.email)
      this.updateData()
    },
    updateData () {
      const db = getDatabase()
      update(ref(db, 'candidates/' + this.selectedData.id), {
        ...this.selectedData
      })
      const data = {
        ...this.selectedData,
        status: this.selectedStatus,
        test: '',
        avatar: '',
        resume: ''
      }
      logger(this.$store.getters.getEmail, 'Обновление данных', data)
    },
    async saveComment () {
      const comment = { ...this.selectedData }

      if (typeof comment.comment === 'string') {
        comment.comment = {}
      }
      const name = this.$store.getters.getEmail.toString().split('@')[0]

      if (comment.comment[name]) {
        delete comment.comment[name]
      }
      if (this.comment) {
        comment.comment[name] = this.comment
      }
      const db = getDatabase()
      await update(ref(db, 'candidates/' + this.selectedData.id), {
        ...comment
      })

      logger(this.$store.getters.getEmail, 'Оставил комментарий #' + comment.id, this.comment)
    },
    async deleteData (id) {
      const db = getDatabase()
      await set(ref(db, 'candidates/' + id), null)
      this.selectedId = 0
      this.loadData()

      logger(this.$store.getters.getEmail, 'Удаление кандидата', id)
    },
    loadData () {
      const dbRef = ref(getDatabase())
      get(child(dbRef, 'vacancies')).then((snapshot) => {
        const data = Object.values(snapshot.val()).map((item) => ({
          item: item.title,
          name: item.title,
          isDel: item.isDel
        }))
        this.jobList = data || []
      })
      get(child(dbRef, 'candidates')).then((snapshot) => {
        if (snapshot.exists()) {
          const data = Object.values(snapshot.val())
          this.list = data || []
        } else {
          console.log('No data available')
        }
      }).catch((error) => {
        console.error(error)
      })
    },
    async saveData (func) {
      if (this.editUserId) {
        const db = getDatabase()
        await update(ref(db, 'candidates/' + this.newUser.id), {
          ...this.newUser
        })
        const data = {
          ...this.newUser,
          test: '',
          avatar: '',
          resume: ''
        }
        logger(this.$store.getters.getEmail, 'Обновление кандидата', data)
      } else {
        const db = getDatabase()
        const id = Number(new Date())
        await set(ref(db, 'candidates/' + id), {
          id: id,
          ...this.newUser,
          hrEmail: this.$store.getters.getEmail
        })

        const data = {
          ...this.newUser,
          status: 'Откликнулся',
          test: '',
          avatar: '',
          resume: ''
        }
        logger(this.$store.getters.getEmail, 'Создание кандидата', data)
      }
      func()
      this.clearData()
      this.loadData()
    },
    saveAvatar (file) {
      const reader = new FileReader()
      reader.readAsDataURL(file)
      reader.onload = () => {
        this.newUser.avatar = reader.result
      }
    },
    saveResume (file) {
      const reader = new FileReader()
      reader.readAsDataURL(file)
      reader.onload = () => {
        this.newUser.resume = reader.result
      }
      const reader2 = new FileReader()
      reader2.readAsText(file)
      reader2.onload = () => {
        this.autoInputData(reader2.result)
      }
    },
    saveTest (file) {
      const reader = new FileReader()
      reader.readAsDataURL(file)
      reader.onload = () => {
        this.selectedData.test = reader.result
      }
    },
    autoInputData (text) {
      try {
        const autoMail = text.match(/:.{1,}@.{1,10}"/).map((item) => item.slice(1, -1))[0]
        if (autoMail.length < 40) {
          this.newUser.email = autoMail || ''
        }
      } catch (e) {
        console.log('empty')
      }

      try {
        const autoPhone = text.match(/\+7.{1,32}/).map((item) => item.replace(/\\uc0\\u[0-9]{3}/g, ''))[0]
        if (autoPhone.length < 20) {
          this.newUser.phone = autoPhone || ''
        }
      } catch (e) {
        console.log('empty')
      }

      try {
        const result = text.match(/\\uc0\\u[0-9]{4}/g).map((item) => item.substr(-4)).map((item) => String.fromCharCode(item))
        const male = ~result.join('').indexOf('Мужчина') ? 'Мужской' : 'Женский'
        this.newUser.male = male
      } catch (e) {
        console.log('empty')
      }

      try {
        const result = text.slice(60000)
          .match(/(.*?)fs18/)[0]
          .match(/\\uc0\\u[0-9]{4}|(\s\s)/g).map((item) => item.substr(-4)).map((item) => {
            if (item === '  ') {
              return ' '
            }
            return String.fromCharCode(item)
          })
          .join('')
        if (result.length < 100) {
          this.newUser.name = result
        }
      } catch (e) {
        console.log('empty')
      }
    },
    clearData () {
      this.editUserId = 0
      this.comment = ''
      this.newUser = {
        avatar: undefined,
        name: '',
        email: '',
        male: '',
        age: '',
        city: '',
        phone: '',
        education: '',
        about: '',
        hrEmail: '',
        status: 1,
        vacation: '',
        comment: '',
        resume: '',
        test: ''
      }
    }
  },
  data: () => ({
    list: [],
    editUserId: 0,
    comment: '',
    jobList: [],
    searchUser: '',
    filterJob: '',
    filterStatus: '',
    selectedId: 0,
    optionsMale: [
      { item: 'Мужской', name: 'Мужской' },
      { item: 'Женский', name: 'Женский' }
    ],
    statusOptions: [
      { item: '1', name: 'Откликнулся' },
      { item: '2', name: 'Тестирование' },
      { item: '3', name: 'Интервью с рекрутером' },
      { item: '4', name: 'Интервью с руководителем' },
      { item: '5', name: 'Проверка ИБ' },
      { item: '6', name: 'Мед. комиссия' },
      { item: '7', name: 'Предложение о работе' },
      { item: '8', name: 'Принят' },
      { item: '9', name: 'Отклонен' }
    ],
    newUser: {
      avatar: undefined,
      name: '',
      email: '',
      male: '',
      age: '',
      city: '',
      phone: '',
      education: '',
      about: '',
      hrEmail: '',
      status: 1,
      vacation: '',
      comment: {},
      resume: '',
      test: ''
    },
    mailText: ''
  })
}
</script>

<style scoped>
.wrapper {
  display: flex;
}

.list {
  height: calc(100vh - 100px);
  width: 30%;
  overflow: auto;
  box-shadow: 6px 1px 13px -3px rgba(34, 60, 80, 0.17);
}

.data {
  height: calc(100vh - 100px);
  padding: 20px;
  overflow: auto;
  width: 100%;
}

.item {
  position: relative;
  display: flex;
  font-size: 13px;
  padding: 5px;
  font-weight: bolder;
  cursor: pointer;
  align-items: center;
  gap: 16px;
  border-bottom: 1px solid #c1c1c1;
  transition: .1s;
}

.item:hover {
  background-color: #faeaff;
}

.buttons {
  display: flex;
  justify-content: flex-end;
  gap: 20px;
}

.add {
  position: absolute;
  bottom: 30px;
  right: 30px;
}

.avatar img {
  width: 100px;
}

.content {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 8px;
}

.head {
  text-align: center;
  width: 100%;
}

.row {
  display: flex;
  gap: 8px;
  width: 100%;
  padding: 0;
  margin: 0;
  flex-direction: row;
}

.row input {
  width: 49%;
}

.form-control {
  display: block;
  width: 100%;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: #212529;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  appearance: none;
  border-radius: 0.25rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.mainInfo {
  display: flex;
  gap: 50px;
  margin-top: 20px;
}

.photo {
  width: 200px;
  border-radius: 10px;
  overflow: hidden;
}
.photo img {
  width: 100%;
}

.name {
  text-align: left;
  font-size: 30px;
  font-weight: bolder;
}

.age {
  text-align: left;
  color: #858585;
}

.userLabel {
  font-weight: bolder;
  margin-right: 8px;
}

.user-about {
  text-align: left;
  margin-top: 16px;
  color: #858585;
}

.searchName {
  margin: 8px;
  width: 95%;
}

.success {
  width: 4px;
  height: 50px;
  position: absolute;
  right: 0;
  background-color: #198754;
}

.fail {
  width: 4px;
  height: 50px;
  position: absolute;
  right: 0;
  background-color: #dc3545;
}

.progress {
  width: 4px;
  height: 50px;
  position: absolute;
  right: 0;
  background-color: #858585;
}
.comment {
  display: flex;
  gap: 12px;
  flex-direction: column;
  align-items: flex-start;
  width: 40%;
}

.filterLabel {
  position: absolute;
  color: #858585;
  font-size: 16px;
  top: 7px;
  pointer-events: none;
  left: 20px;
}

.isDelUser {
  background-color: #dddb0c;
}
</style>
