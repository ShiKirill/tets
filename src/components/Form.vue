<template>
  <div class="main_block">
    <form
      class="form_block"
      @submit.prevent="onSubmit"
      v-if="dataForRender && readyForRequest"
    >
      <div
        class="form_field"
        v-for="item in dataForRender"
        :key="dataForRender.indexOf(item)"
        :class="{ form_field_checkbox: item.type === 'checkbox' }"
      >
        <label
          :for="item.name"
          @click="onCheck(item.name)"
          class="form_field__label"
          >{{ item.title }}</label
        >
        <imask-input
          class="form_field__input"
          :id="item.name"
          :lazy="false"
          :class="{
            'form_field__input_error':
              !$v.validationData[item.name].required &&
              $v.validationData[item.name].$error,
          }"
          v-model="validationData[item.name]"
          @change="$v.validationData[item.name].$touch()"
          :mask="'+{7} 000 000 00 00'"
          v-if="item.name === 'phone'"
        />
        <div
          v-else-if="item.name === 'select'"
          class="form_field__input select"
          :class="{ 'show-options': showOptions }"
        >
          <img
            src="../assets/arrow.svg"
            alt="arrow down"
            @click="optionShow"
            class="select__icon"
          />
          <div class="select__list">
            <input
              name="option"
              class="form_field__input select__list__option"
              @click="optionShow"
              :class="{ 'first-option': showOptions }"
              :value="validationData.optionValue"
              readonly
            />
            <div
              class="form_field__input select__list__option"
              @click="optionClick"
            >
              Administrator
            </div>
            <div
              class="form_field__input select__list__option"
              @click="optionClick"
            >
              Executor
            </div>
            <div
              class="form_field__input select__list__option"
              @click="optionClick"
            >
              Applicant
            </div>
          </div>
        </div>
        <div v-else-if="item.name === 'checkbox'" class="checkbox">
          <input
            type="checkbox"
            @input="onCheck(item.name)"
            class="checkbox__input"
          />
          <span
            class="checkbox__span"
            :class="{
              checkbox__span_checked: validationData.checkbox,
              checkbox__span_disabled: disabledCheckbox,
            }"
            ><svg
              v-if="validationData.checkbox"
              class="checked-icon"
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
          @input="$v.validationData[item.name].$touch()"
          :id="item.name"
          class="form_field__input"
          :class="{
            'form_field__input_error':
              !$v.validationData[item.name].required &&
              $v.validationData[item.name].$error,
          }"
        />
        <div
          class="error"
          v-if="
            (!$v.validationData[item.name].required &&
              $v.validationData[item.name].$error)
          "
        >
          Field is required
        </div>
      </div>
      <button type="submit" class="form_block__button">
        Зарегистрироваться
      </button>
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
        optionValue: 'Administrator',
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
        required: (val)=>{
          return val.replace('_', '').length === 16;
        }
      },
      checkbox: {
        required: (val)=>{
          return val
        }
      },
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
      if (name !== 'checkbox') return;
      this.validationData.checkbox = !this.validationData.checkbox;
      this.$v.validationData[name].$touch();
    },
    onSubmit() {
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

<style lang="scss" scoped>
.main_block {
  padding: 4.3125rem 0;
  background: #f2f4f5;
  width: 50%;
  height: 100%;
  overflow-y: auto;
  box-sizing: border-box;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;

  &::-webkit-scrollbar {
    width: 0;
  }

  @media screen and (max-width: 800px) {
    width: 100%;
  }
}

.form_block {
  width: 55.8%;
  min-width: 23.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #ffffff;
  border: 1px solid #e6e9f0;
  box-sizing: border-box;
  border-radius: 0.5rem;
  padding: 2rem;

  .form_block__button {
    margin-top: 3.875rem;
    width: 66.4%;
    height: 2.25rem;
    background: #344887;
    border-radius: 0.25rem;
    border: none;
    cursor: pointer;
    line-height: 1.125rem;
    text-transform: uppercase;
    color: #ffffff;
    padding: 0.5rem 1rem;
    transition: all 0.3s;
    font: {
      weight: 500;
      size: 0.875rem;
    }

    &:hover {
      border-color: #344887;
      background: linear-gradient(
          0deg,
          rgba(196, 209, 255, 0.1),
          rgba(196, 209, 255, 0.1)
        ),
        #344887;
    }

    &:active {
      background: #7b61ff;
      border: 1px dashed #7b61ff;
    }

    &:disabled {
      border-color: rgba(119, 119, 119, 0.1);
      background: rgba(119, 119, 119, 0.1);
      color: #999999;
    }
  }

  @media screen and (max-width: 800px) {
    width: 60%;
    min-width: auto;
  }

  @media screen and (max-width: 550px) {
    width: 75%;
    .form_block__button {
      width: 85%;
    }
  }
}

.form_field {
  margin-top: 1.5rem;
  position: relative;
  width: 100%;
  display: flex;
  flex-direction: column;
  line-height: 1.5rem;
  font: {
    size: 1rem;
    style: normal;
    weight: normal;
  }

  &:first-child {
    margin-top: 0;
  }

  .form_field__label {
    width: fit-content;
    letter-spacing: 0.005rem;
    color: #85868c;
    font-feature-settings: 'liga' off;
  }

  .form_field__input {
    height: 3.5rem;
    width: 100%;
    margin-top: 0.5rem;
    border: 1px solid #cad0e0;
    box-sizing: border-box;
    border-radius: 0.25rem;
    padding: 0 0.75rem;
    line-height: 1.125rem;
    color: #212121;
    font: {
      family: Geometria, sans-serif;
      size: 1rem;
    }

    &:focus {
      outline: none;
      border-color: rgb(0, 0, 119);
    }

    &:disabled {
      border-color: #999999;
      background: rgba(119, 119, 119, 0.1);
    }
  }

  .error {
    position: absolute;
    bottom: -1.25rem;
    font-size: 0.75rem;
    line-height: 0.75rem;
    letter-spacing: 0.004rem;
    color: #ef3e4a;
  }
}

.form_field_checkbox {
  flex-direction: row-reverse;
  justify-content: flex-end;
  align-items: center;

  .form_field__label {
    cursor: pointer;
  }

  .checkbox {
    padding: 0;
    margin: 0;
    position: relative;
    margin-right: 0.6875rem;
    border: none;

    .checkbox__input {
      position: absolute;
      left: 0;
      top: 0;
      width: 110%;
      height: 110%;
      z-index: 2;
      opacity: 0;
      margin: 0;
      cursor: pointer;
    }

    .checkbox__span {
      display: block;
      width: 1rem;
      height: 1rem;
      z-index: 1;
      outline: 2px solid #6b7182;
      border-radius: 0.125rem;

      .checked-icon {
        width: 112.5%;
        height: 112.5%;
        position: absolute;
        left: -0.0625rem;
        top: -0.0625rem;
      }

      .hover-icon {
        position: absolute;
        top: 0;
        left: 0;
        transform: translate(-30%, -30%);
        opacity: 0;
        transition: opacity 0.5s;
        z-index: 1;
        cursor: context-menu;
      }
    }

    &:hover {
      .hover-icon {
        opacity: 1;
        height: 2.5rem;
        width: 2.5rem;
      }
    }

    .checkbox__span_checked {
      outline: none;
    }

    .checkbox__span_disabled {
      outline: 2px solid #999999;

      .checked-icon path {
        fill: #999999;
      }
    }
  }
}

.select {
  border: none !important;
  position: relative;
  background: #f0f0f0;
  overflow: hidden;

  .select__icon {
    z-index: 3;
    cursor: pointer;
    position: absolute;
    right: 1.1875rem;
    top: 47%;
  }

  .select__list {
    width: 100%;
    border-radius: 0.1875rem;
    position: absolute;
    left: 0;
    z-index: 2;

    .select__list__option {
      cursor: pointer;
      margin-top: 0;
      display: flex;
      align-items: center;
      background: #fff;

      &:focus {
        border-color: #cad0e0;
      }
    }
  }
}

.show-options {
  overflow: inherit;

  .select__icon {
    transform: rotate(180deg);
  }
}

.form_field__input_error {
  background: rgba(243, 39, 53, 0.08);
  border: 1px solid #ef3e4a !important;
}

.success-message {
  width: 55.8%;
  min-width: 23.5rem;
  display: flex;
  height: 4.75rem;
  justify-content: center;
  align-items: center;
  background: #ffffff;
  border: 1px solid #e6e9f0;
  box-sizing: border-box;
  border-radius: 0.5rem;
  
  @media screen and (max-width: 800px){
    min-width: 10rem;
  }
}
</style>
