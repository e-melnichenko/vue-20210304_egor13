<template>
  <form class="form meetup-form" @submit.prevent="handleSubmit">
    <div class="meetup-form__content">
      <fieldset class="form-section">
        <div class="form-group">
          <label>Название</label>
          <input class="form-control" v-model="localMeetup.title"/>
        </div>
        <div class="form-group">
          <label>Дата</label>
          <input class="form-control" type="date" v-model="localMeetup.date"/>
        </div>
        <div class="form-group">
          <label>Место</label>
          <input class="form-control" v-model="localMeetup.place"/>
        </div>
        <div class="form-group">
          <label>Описание</label>
          <textarea class="form-control" rows="3" v-model="localMeetup.description"></textarea>
        </div>
        <div class="form-group">
          <label>Изображение</label>
          <image-uploader v-model="localMeetup.imageId"/>
        </div>
      </fieldset>

      <h3 class="form__section-title">Программа</h3>
      <meetup-agenda-item-form
        class="mb-3"
        v-for="(agendaItem, index) in localMeetup.agenda"
        :key="agendaItem.id"
        :agendaItem.sync="localMeetup.agenda[index]"
        @remove="() => handleRemove(index)"
      />

      <div class="form-section_append">
        <button type="button" data-test="addAgendaItem" @click="addAgendaItem">+ Добавить этап программы</button>
      </div>
    </div>

    <div class="meetup-form__aside">
      <div class="meetup-form__aside_stick">
        <!-- data-test атрибуты используются для поиска нужного элемента в тестах, не удаляйте их -->
        <button class="button button_secondary button_block" type="button" data-test="cancel" @click="$emit('cancel')">Отмена</button>
        <button class="button button_primary button_block" type="submit" data-test="submit">{{ submitText }}</button>
      </div>
    </div>
  </form>
</template>

<script>
import MeetupAgendaItemForm from './MeetupAgendaItemForm.vue';
import ImageUploader from './ImageUploader';
// Jest ругается на этот импорт
// import { cloneDeep } from 'lodash-es';

let lastId = -1;
function createAgendaItem() {
  return {
    id: lastId--,
    startsAt: '00:00',
    endsAt: '00:00',
    type: 'other',
    title: null,
    description: null,
    speaker: null,
    language: null,
  };
}

export default {
  name: 'MeetupForm',

  components: {
    ImageUploader,
    MeetupAgendaItemForm,
  },

  props: {
    meetup: {
      type: Object,
      required: true,
    },
    submitText: String,
  },

  data() {
    return {
      localMeetup: JSON.parse(JSON.stringify(this.meetup)),
      // localMeetup: cloneDeep(this.meetup),
    }
  },

  methods: {
    handleSubmit() {
      this.$emit('submit', {...this.localMeetup})
    },

    addAgendaItem() {
      const agendaItem = createAgendaItem();
      const {agenda} = this.localMeetup;

      if (agenda.length > 0) {
        agendaItem.startsAt = agenda[agenda.length-1].endsAt;
      }

      agenda.push(agendaItem)
    },

    handleRemove(index) {
      this.localMeetup.agenda.splice(index, 1);
    }
  },
};
</script>

<style scoped>
.meetup-form__aside {
  margin: 48px 0;
}

.meetup-form__aside_stick > .button {
  margin: 0 0 12px 0;
}

@media all and (min-width: 992px) {
  .meetup-form {
    display: flex;
    flex-direction: row;
  }

  .meetup-form__content {
    flex: 1 0 calc(100% - 320px);
  }

  .meetup-form__aside {
    flex: 1 0 320px;
    max-width: 320px;
    width: 100%;
    padding-left: 137px;
    margin: 0;
  }

  .meetup-form__aside_stick {
    position: sticky;
    top: 32px;
  }
}
</style>
