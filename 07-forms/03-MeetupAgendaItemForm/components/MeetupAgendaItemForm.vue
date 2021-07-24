<template>
  <div class="form-section form-section_inner">
    <button type="button" class="remove-button" @click="$emit('remove')">
      <app-icon icon="trash" />
    </button>

    <div class="form-group">
      <select
        class="form-control"
        title="Тип"
        @change="($event) => updateAgendaItem('type', $event)"
      >
        <option
          v-for="type in agendaTypes"
          :value="type.value"
          :key="type.value"
          :selected="type.value === localAgendaItem.type"
        >
          {{ type.text }}
        </option>
      </select>
    </div>

    <div class="form__row">
      <div class="form__col">
        <div class="form-group">
          <label class="form-label">Начало</label>
          <input
            class="form-control" type="time" placeholder="00:00"
            :value="localAgendaItem.startsAt"
            @input="handleStartsAtChange"
          />
        </div>
      </div>
      <div class="form__col">
        <div class="form-group">
          <label class="form-label">Окончание</label>
          <input class="form-control" type="time" placeholder="00:00"
            :value="localAgendaItem.endsAt"
            @input="($event) => updateAgendaItem('endsAt', $event)"
          />
        </div>
      </div>
    </div>

    <div class="form-group">
      <label class="form-label">{{ titleLabel }}</label>
      <input
        class="form-control"
        :value="localAgendaItem.title"
        @input="($event) => updateAgendaItem('title', $event)"
      />
    </div>

    <div class="form-group" v-if="localAgendaItem.type === 'talk'">
      <label class="form-label">Докладчик</label>
      <input
        class="form-control"
        :value="localAgendaItem.speaker"
        @input="($event) => updateAgendaItem('speaker', $event)"
      />
    </div>

    <div
      class="form-group"
      v-if="localAgendaItem.type === 'talk' || localAgendaItem.type === 'other'"
    >
      <label class="form-label">Описание</label>
      <textarea
        class="form-control"
        :value="localAgendaItem.description"
        @input="($event) => updateAgendaItem('description', $event)"
      ></textarea>
    </div>
    <div class="form-group" v-if="localAgendaItem.type === 'talk'">
      <label class="form-label">Язык</label>
      <select
        class="form-control"
        @change="($event) => updateAgendaItem('language', $event)"
      >
        <option
          v-for="language in talkLanguages"
          :key="language.value"
          :value="language.value"
        >
          {{ language.text }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
import AppIcon from './AppIcon';

const getAgendaItemTypes = () => [
  { value: 'registration', text: 'Регистрация' },
  { value: 'opening', text: 'Открытие' },
  { value: 'break', text: 'Перерыв' },
  { value: 'coffee', text: 'Coffee Break' },
  { value: 'closing', text: 'Закрытие' },
  { value: 'afterparty', text: 'Afterparty' },
  { value: 'talk', text: 'Доклад' },
  { value: 'other', text: 'Другое' },
];

const getTalkLanguages = () => [
  { value: null, text: 'Не указано' },
  { value: 'RU', text: 'RU' },
  { value: 'EN', text: 'EN' },
];

export default {
  name: 'MeetupAgendaItemForm',

  components: { AppIcon },

  props: {
    agendaItem: {
      type: Object,
      required: true
    }
  },

  data() {
    return {
      localAgendaItem: {...this.agendaItem},
      currentStartsAt: null,
    }
  },

  computed: {
    agendaTypes() {
      return getAgendaItemTypes()
    },

    talkLanguages() {
      return getTalkLanguages()
    },

    titleLabel() {
      return this.localAgendaItem.type === 'talk' ? 'Тема' :
        this.localAgendaItem.type === 'other' ? 'Заголовок' : 'Нестандартный текст (необязательно)'
    }
  },

  methods: {
    updateAgendaItem(field, $event) {
      this.localAgendaItem[field] = $event.target.value;
      this.$emit('update:agendaItem', {...this.localAgendaItem})
    },

    handleStartsAtChange($event) {
      // $event.target.valueAsNumber не проходит в тестах, тк в node нет valueAsNumber

      const numValue = this.getNumberFromTimeString($event.target.value);
      const delta = numValue - this.currentStartsAt;

      const endsAtNumber = this.getNumberFromTimeString(this.localAgendaItem.endsAt);
      const date = new Date(endsAtNumber + delta);
      const hhmm = date.toISOString().slice(11, 16);

      this.currentStartsAt = numValue;
      this.localAgendaItem.startsAt = $event.target.value;
      this.localAgendaItem.endsAt = hhmm;

      this.$emit('update:agendaItem', {
        ...this.localAgendaItem,
      })
    },

    getNumberFromTimeString(time) {
      const [hours, minutes] = time.split(':');
      const date = new Date(0);
      date.setUTCHours(+hours);
      date.setUTCMinutes(+minutes);

      return +date
    }
  },

  mounted() {
    this.currentStartsAt = this.getNumberFromTimeString(this.localAgendaItem.startsAt)
  }
};
</script>

<style></style>
