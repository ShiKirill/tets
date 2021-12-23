<template>
  <div class="main-block" :class="{ 'after-request': !readyForRequest }">
    <form
      class="form-block"
      @submit="onSubmit"
      v-if="dataForRender && readyForRequest"
    >
      <div
        class="form-input-block"
        v-for="item in dataForRender"
        :key="dataForRender.indexOf(item)"
        :class="{ 'input-block-checkbox': item.type == 'checkbox' }"
      >
        <label
          :for="item.name"
          @click="onCheck(item.name)"
          class="form-input-label"
          :class="{ 'checkbox-cursor': item.type == 'checkbox' }"
          >{{ item.title }}</label
        >
        <imask-input
          class="form-input-item"
          :class="{
            'block-error':
              !$v.validationData[item.name].required &&
              $v.validationData[item.name].$error,
          }"
          v-model="validationData.phone"
          :mask="'+{7} 000 000 00 00'"
          v-if="item.name == 'phone'"
        />
        <div
          v-else-if="item.name == 'select'"
          class="select form-input-item"
          :class="{ 'show-options': showOptions }"
        >
          <img
            src="../assets/arrow.svg"
            alt="arrow down"
            @click="optionShow"
            class="select-arrow"
          />
          <div class="select-options">
            <input
              name="option"
              class="option form-input-item"
              @click="optionShow"
              :class="{ 'first-option': showOptions }"
              :value="validationData.optionValue"
              readonly
            />
            <div class="option form-input-item" @click="optionClick">
              Администратор
            </div>
            <div class="option form-input-item" @click="optionClick">
              Исполнитель
            </div>
            <div class="option form-input-item" @click="optionClick">
              Заявитель
            </div>
          </div>
        </div>
        <div
          v-else-if="item.name == 'checkbox'"
          class="form-input-item checkbox checkbox-cursor"
        >
          <input
            type="checkbox"
            @input="onCheck(item.name)"
            class="checkbox-input"
          />
          <span
            class="checkbox-span"
            :class="{
              'checkbox-no-border': validationData.checkbox,
              'disabled-checkbox': disabledCheckbox,
            }"
            ><svg
              v-if="validationData.checkbox"
              width="18"
              height="18"
              viewBox="0 0 18 18"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M7 14L2 9L3.41 7.58L7 11.17L14.59 3.58L16 5L7 14ZM16 0H2C0.89 0 0 0.89 0 2V16C0 16.5304 0.210714 17.0391 0.585786 17.4142C0.960859 17.7893 1.46957 18 2 18H16C16.5304 18 17.0391 17.7893 17.4142 17.4142C17.7893 17.0391 18 16.5304 18 16V2C18 0.89 17.1 0 16 0Z"
                fill="#344887"
              />
            </svg>
            <svg
              v-if="!disabledCheckbox"
              class="hover-icon"
              width="40"
              height="40"
              viewBox="0 0 40 40"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <rect
                width="40"
                height="40"
                rx="20"
                transform="matrix(1 0 0 -1 0 40)"
                fill="#777777"
                fill-opacity="0.1"
              />
            </svg>
          </span>
        </div>
        <input
          v-else
          v-model="validationData[item.name]"
          :type="item.type"
          @input="$v.validationData.$touch()"
          :id="item.name"
          class="form-input-item"
          :class="{
            'block-error':
              !$v.validationData[item.name].required &&
              $v.validationData[item.name].$error,
          }"
        />
        <div
          class="error"
          v-if="
            (!$v.validationData[item.name].required &&
              $v.validationData[item.name].$error) ||
            (!validationData.checkbox && item.name == 'checkbox')
          "
        >
          Field is required
        </div>
      </div>
      <button type="submit" class="form-btn">Зарегистрироваться</button>
    </form>
    <div class="success-message" v-else-if="!readyForRequest">Успешно</div>
  </div>
</template>

<script>
import axios from 'axios';
import MockAdapter from 'axios-mock-adapter';
import { required } from 'vuelidate/lib/validators';
import { IMaskComponent } from 'vue-imask';

export default {
  name: 'HelloWorld',
  data() {
    return {
      readyForRequest: true,
      disabledCheckbox: false,
      data: '',
      showOptions: false,
      validationData: {
        email: '',
        name: '',
        surname: '',
        phone: '',
        checkbox: '',
        optionValue: 'Администратор',
      },
    };
  },
  validations: {
    validationData: {
      email: {
        required,
      },
      name: {
        required,
      },
      surname: {
        required,
      },
      phone: {
        required,
      },
      checkbox: {},
      select: {},
    },
  },
  components: {
    'imask-input': IMaskComponent,
  },
  computed: {
    dataForRender() {
      if (this.data) {
        return this.data;
      } else {
        return '';
      }
    },
  },
  created() {
    const mock = new MockAdapter(axios);
    mock.onGet('/api/form').reply(200, [
      {
        type: 'email',
        title: 'Email',
        name: 'email',
        required: true,
      },
      {
        type: 'string',
        title: 'Name',
        required: true,
        name: 'name',
      },
      {
        type: 'string',
        title: 'Surname',
        required: true,
        name: 'surname',
      },
      {
        type: 'string',
        title: 'Phone',
        name: 'phone',
        required: true,
      },
      {
        type: 'select',
        title: 'Select role',
        name: 'select',
      },
      {
        type: 'checkbox',
        title: 'I agree to the processing of personal data',
        name: 'checkbox',
        required: true,
      },
    ]);
    mock.onPost().reply(200);
    axios.get('/api/form').then((resp) => {
      this.data = resp.data;
    });
  },
  methods: {
    onCheck(name) {
      if (name != 'checkbox') return;
      this.validationData.checkbox = !this.validationData.checkbox;
    },
    onSubmit(e) {
      e.preventDefault();
      this.$v.validationData.$touch();
      if (!this.$v.$anyError && this.validationData.checkbox) {
        axios.post('/api/form', this.$v.validationData.$model).then(() => {
          this.readyForRequest = false;
        });
      }
    },
    optionShow() {
      this.showOptions = !this.showOptions;
    },
    optionClick(e) {
      this.validationData.optionValue = e.target.innerText;
      this.showOptions = !this.showOptions;
    },
  },
};
</script>

<style scoped>
@import url('../assets/fonts/Geometria/stylesheet.css');
.main-block {
  padding: 4.3125rem 0;
  background: #f2f4f5;
  width: 50%;
  height: 100%;
  overflow-y: auto;
  box-sizing: border-box;
}
.form-block {
  margin: 0 auto;
  width: 55.8%;
  min-width: 23.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #ffffff;
  border: 1px solid #e6e9f0;
  box-sizing: border-box;
  border-radius: 8px;
  padding: 2rem;
}
.form-input-block {
  position: relative;
  width: 100%;
  display: flex;
  flex-direction: column;
}
.form-input-block {
  margin-top: 1.5rem;
}
.form-input-block:first-child {
  margin-top: 0;
}
.form-input-label {
  width: fit-content;
  font-family: Geometria;
  font-style: normal;
  font-weight: normal;
  font-size: 16px;
  line-height: 24px;
  letter-spacing: 0.005em;
  font-feature-settings: 'liga' off;
  color: #85868c;
}
.form-input-item {
  height: 3.5rem;
  width: 100%;
  margin-top: 0.5rem;
  border: 1px solid #cad0e0;
  box-sizing: border-box;
  border-radius: 4px;
  padding: 0 0.75rem;
  font-family: Geometria;
  font-style: normal;
  font-weight: normal;
  font-size: 16px;
  line-height: 18px;
  color: #212121;
}
.form-input-item:focus {
  outline: none;
}
.input-block-checkbox {
  flex-direction: row-reverse;
  justify-content: flex-end;
  align-items: center;
}
.checkbox {
  padding: 0;
  margin: 0;
  position: relative;
  margin-right: 0.6875rem;
  width: 1.25rem;
  height: 1.125rem;
  border: none;
}
.checkbox-cursor {
  cursor: pointer;
}
.checkbox-input {
  cursor: pointer;
  position: absolute;
  left: 0;
  top: 0;
  width: 110%;
  height: 110%;
  z-index: 2;
  opacity: 0;
  margin: 0;
}
.checkbox-span {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  outline: 2px solid #6b7182;
  border-radius: 5px;
}
.checkbox-no-border {
  outline: none;
}
.checkbox-span svg {
  width: 100%;
  height: 100%;
}
.hover-icon {
  width: auto;
  height: auto;
  position: absolute;
  top: 0;
  left: 0;
  transform: translate(-28%, -28%);
  display: none;
  z-index: 1;
  cursor: context-menu;
}
.checkbox:hover .hover-icon {
  display: inherit;
  height: 40px;
  width: 40px;
}
.disabled-checkbox {
  outline: 2px solid #999999;
}
.disabled-checkbox svg path {
  fill: #999999;
}
.form-btn {
  margin-top: 3.875rem;
  width: 66.4%;
  height: 2.25rem;
  background: #344887;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-family: Geometria;
  font-style: normal;
  font-weight: 500;
  font-size: 14px;
  line-height: 20px;
  text-transform: uppercase;
  color: #ffffff;
  padding: 0.5rem 1rem;
}
.select {
  border: none;
  position: relative;
  background: #f0f0f0;
  overflow: hidden;
}
.select-options {
  width: 100%;
  border-radius: 3px;
  position: absolute;
  left: 0;
  z-index: 2;
}
.show-options {
  overflow: inherit;
}
.select-arrow {
  z-index: 3;
  cursor: pointer;
  position: absolute;
  right: 1.1875rem;
  top: 50%;
  transform: translateY(-50%);
}
.option {
  cursor: pointer;
  margin-top: 0;
  display: flex;
  align-items: center;
  background: #fff;
}
.error {
  position: absolute;
  bottom: -1.25rem;
  font-family: Geometria;
  font-style: normal;
  font-weight: normal;
  font-size: 12px;
  line-height: 12px;
  letter-spacing: 0.004em;
  color: #ef3e4a;
}
.block-error {
  background: rgba(243, 39, 53, 0.08);
  border: 1px solid #ef3e4a;
}
input:focus {
  border-color: rgb(0, 0, 119);
}
input:disabled {
  border-color: #999999;
  background: rgba(119, 119, 119, 0.1);
}
.success-message {
  width: 55.8%;
  min-width: 23.5rem;
  display: flex;
  height: 76px;
  margin: 0 auto;
  justify-content: center;
  align-items: center;
  background: #ffffff;
  border: 1px solid #e6e9f0;
  box-sizing: border-box;
  border-radius: 8px;
}
.after-request {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
