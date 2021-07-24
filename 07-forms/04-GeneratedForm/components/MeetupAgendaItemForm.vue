<template>
  <DIV class="form-section form-section_inner">
    <button type="button" class="remove-button" @click="$emit('remove')">
      <app-icon icon="trash"/>
    </button>

    <form-group>
      <dropdown-button
        title="Тип"
        :value="localAgendaItem.type"
        :options="$options.agendaItemTypeOptions"
        @change="(value) => updateAgendaItem('type', value)"
      />
    </form-group>

    <div class="form__row">
      <div class="form__col">
        <form-group label="Начало">
          <app-input
            type="time"
            placeholder="00:00"
            :value="localAgendaItem.startsAt"
            @input="handleStartsAtChange"
          />
        </form-group>
      </div>
      <div class="form__col">
        <form-group label="Окончание">
          <app-input
            type="time"
            placeholder="00:00"
            :value="localAgendaItem.endsAt"
            @input="(value) => updateAgendaItem('endsAt', value)"
          />
        </form-group>
      </div>
    </div>

    <form-group
      v-for="item in $options.fieldSpecifications[localAgendaItem.type]"
      :key="item.field"
      :label="item.title"
    >
      <component
        :is="item.component"
        v-bind="item.props"
        :[item.model.prop]="localAgendaItem[item.field]"
        @[item.model.event]="(value) => updateAgendaItem(item.field, value)"
      />
    </form-group>
  </DIV>
</template>

<script>
import AppInput from './AppInput';
import DropdownButton from './DropdownButton';
import AppIcon from './AppIcon';
import FormGroup from './FormGroup';
import {
  getAgendaItemsFieldSpecifications,
  getAgendaItemTypeOptions,
  getAgendaItemLanguageOptions,
} from '../MeetupService';

export default {
  name: 'MeetupAgendaItemForm',

  components: { FormGroup, AppIcon, AppInput, DropdownButton },

  props: {
    agendaItem: {
      type: Object,
      required: true,
    }
  },

  data() {
    return {
      localAgendaItem: {...this.agendaItem},
      currentStartsAt: null,
    }
  },

  methods: {
    updateAgendaItem(field, value) {
      this.localAgendaItem[field] = value;

      this.$emit('update:agendaItem', {
        ...this.localAgendaItem,
      });
    },

    handleStartsAtChange(value) {
      // $event.target.valueAsNumber не проходит в тестах, тк в node нет valueAsNumber

      const numValue = this.getNumberFromTimeString(value);
      const delta = numValue - this.currentStartsAt;

      const endsAtNumber = this.getNumberFromTimeString(this.localAgendaItem.endsAt);
      const date = new Date(endsAtNumber + delta);
      const hhmm = date.toISOString().slice(11, 16);

      this.currentStartsAt = numValue;
      this.localAgendaItem.startsAt = value;
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
  },

  agendaItemTypeOptions: getAgendaItemTypeOptions(),
  fieldSpecifications: getAgendaItemsFieldSpecifications(),
  languageOptions: getAgendaItemLanguageOptions(),
};
</script>

<style></style>
