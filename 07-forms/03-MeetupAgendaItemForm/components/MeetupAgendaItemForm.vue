<template>
  <fieldset class="agenda-item-form">
    <button @click="$emit('remove')" type="button" class="agenda-item-form__remove-button">
      <ui-icon icon="trash" />
    </button>

    <ui-form-group>
      <ui-dropdown title="Тип"
                   v-model="localAgendaItem.type"
                   :options="$options.agendaItemTypeOptions"
                   name="type" />
    </ui-form-group>

    <div class="agenda-item-form__row">
      <div class="agenda-item-form__col">
        <ui-form-group label="Начало">
          <ui-input v-model="startsAt"
                    type="time" placeholder="00:00" name="startsAt" />
        </ui-form-group>
      </div>
      <div class="agenda-item-form__col">
        <ui-form-group label="Окончание">
          <ui-input v-model="localAgendaItem.endsAt"
                    type="time" placeholder="00:00" name="endsAt" />
        </ui-form-group>
      </div>
    </div>

    <ui-form-group :label="titleLabel">
      <ui-input name="title"
                v-model.lazy="localAgendaItem.title" />
    </ui-form-group>
    <ui-form-group v-if="['talk'].includes(type)"
                   label="Докладчик">
      <ui-input name="speaker"
                v-model.lazy="localAgendaItem.speaker" />
    </ui-form-group>
    <ui-form-group v-if="['talk', 'other'].includes(type)"
                   label="Описание">
      <ui-input multiline name="description"
                v-model.lazy="localAgendaItem.description" />
    </ui-form-group>
    <ui-form-group v-if="['talk'].includes(type)"
                   label="Язык">
      <ui-dropdown title="Язык"
                   v-model="localAgendaItem.language"
                   :options="$options.talkLanguageOptions"
                   name="language" />
    </ui-form-group>
  </fieldset>
</template>

<script>
import UiIcon from './UiIcon';
import UiFormGroup from './UiFormGroup';
import UiInput from './UiInput';
import UiDropdown from './UiDropdown';
import dayjs from 'dayjs';

const agendaItemTypeIcons = {
  registration: 'key',
  opening: 'cal-sm',
  talk: 'tv',
  break: 'clock',
  coffee: 'coffee',
  closing: 'key',
  afterparty: 'cal-sm',
  other: 'cal-sm',
};

const agendaItemDefaultTitles = {
  registration: 'Регистрация',
  opening: 'Открытие',
  break: 'Перерыв',
  coffee: 'Coffee Break',
  closing: 'Закрытие',
  afterparty: 'Afterparty',
  talk: 'Доклад',
  other: 'Другое',
};

const agendaItemTypeOptions = Object.entries(agendaItemDefaultTitles).map(([type, title]) => ({
  value: type,
  text: title,
  icon: agendaItemTypeIcons[type],
}));

const talkLanguageOptions = [
  { value: null, text: 'Не указано' },
  { value: 'RU', text: 'RU' },
  { value: 'EN', text: 'EN' },
];

export default {
  name: 'MeetupAgendaItemForm',

  agendaItemTypeOptions,
  talkLanguageOptions,

  components: { UiIcon, UiFormGroup, UiInput, UiDropdown },

  props: {
    agendaItem: {
      type: Object,
      required: true,
    },
  },

  emits: ['update:agendaItem', 'remove'],

  data() {
    return {
      localAgendaItem: { ...this.agendaItem },
    }
  },

  watch: {
    localAgendaItem: {
      handler() {
        this.$emit('update:agendaItem', this.localAgendaItem);
      },
      deep: true,
    },
  },

  computed: {
    type() {
      return this.localAgendaItem.type;
    },

    startsAt: {
      get() {
        return this.localAgendaItem.startsAt;
      },
      set(value) {
        const now = dayjs();
        const getDate = (at) => {
          const [hour, minute] = at.split(':');
          return now.clone().hour(hour).minute(minute);
        };

        const start = getDate(this.localAgendaItem.startsAt);
        const plus = getDate(value);
        const end = getDate(this.localAgendaItem.endsAt);

        this.localAgendaItem.startsAt = value;
        this.localAgendaItem.endsAt = dayjs(end + (plus - start)).format('HH:mm');
      }
    },

    titleLabel() {
      const type = this.type;

      if (type === 'talk') return 'Тема';
      if (type === 'other') return 'Заголовок';
      return 'Нестандартный текст (необязательно)';
    },
  },
};
</script>

<style scoped>
.agenda-item-form {
  border: 2px solid var(--blue-light);
  border-radius: 8px;
  position: relative;
  padding: 20px 10% 0 16px;
}

.agenda-item-form__remove-button {
  position: absolute;
  top: 4px;
  right: 0;
  box-shadow: none;
  border: none;
  background-color: transparent;
  outline: none;
  padding: 4px;
  cursor: pointer;
  transition: 0.2s opacity;
}

.agenda-item-form__remove-button:hover {
  opacity: 0.6;
}

.agenda-item-form__row {
  display: flex;
  flex-direction: column;
}

.agenda-item-form__col + .agenda-item-form__col {
  margin-top: 16px;
}

.agenda-item-form__col:first-child {
  margin-left: 0;
}

@media all and (min-width: 992px) {
  .agenda-item-form {
    padding: 28px 25% 4px 24px;
  }

  .agenda-item-form__remove-button {
    top: 20px;
    right: 20px;
  }

  .agenda-item-form__row {
    flex-direction: row;
    justify-content: space-between;
    margin: 0 -12px;
  }

  .agenda-item-form__col {
    flex: 1 1 auto;
    padding: 0 12px;
  }

  .agenda-item-form__col + .agenda-item-form__col {
    margin-top: 0;
  }

  .agenda-item-form__col:first-child {
    margin-left: 0;
  }
}
</style>
