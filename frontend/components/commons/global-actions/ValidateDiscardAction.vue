<template>
  <div :class="[selectedRecords.length ? '' : 'global-actions--disabled']">
    <ReCheckbox v-model="allSelected" class="list__item__checkbox"></ReCheckbox>
    <slot v-if="position == 'before'" :selectedRecords="selectedRecords" />
    <ReButton class="global-actions__button" @click="onValidate"
      >Validate</ReButton
    >
    <ReButton class="global-actions__button" @click="onDiscard"
      >Discard</ReButton
    >
    <slot v-if="position == 'after'" :selectedRecords="selectedRecords" />
    <p v-if="selectedRecords.length" class="global-actions__text">
      Actions will apply to the
      <span>{{ selectedRecords.length }} records</span> selected
    </p>
  </div>
</template>
<script>
import { mapActions } from "vuex";
import "assets/icons/export";
import "assets/icons/plus";
import "assets/icons/refresh";

export default {
  props: {
    dataset: {
      type: Object,
      required: true,
    },
    position: {
      type: String,
      default: "before",
    },
  },
  data: () => ({
    allSelected: false,
  }),
  computed: {
    visibleRecords() {
      return this.dataset.visibleRecords;
    },
    selectedRecords() {
      // Return selected records.
      return this.visibleRecords.filter((record) => record.selected);
    },
  },
  watch: {
    visibleRecords(newValue) {
      if (!newValue.every((record) => record.selected)) {
        this.allSelected = false;
      }
    },
    allSelected(allSelected) {
      if (
        allSelected ||
        this.visibleRecords.every((record) => record.selected)
      ) {
        this.updateRecords({
          dataset: this.dataset,
          records: this.visibleRecords.map((record) => {
            return { ...record, selected: this.allSelected };
          }),
        });
      }
    },
  },
  methods: {
    ...mapActions({
      updateRecords: "entities/datasets/updateRecords",
    }),
    onDiscard() {
      this.$emit("discard-records", this.selectedRecords);
    },
    async onValidate() {
      this.$emit("validate-records", this.selectedRecords);
    },
  },
};
</script>
<style lang="scss" scoped>
.global-actions {
  &__button {
    border-radius: 5px;
    height: 33px;
    border: none;
    min-width: 80px;
    margin-left: 1em;
    margin-right: 1em;
    outline: none;
    font-weight: 600;
    color: $font-secondary;
    background: $lighter-color;
    border: 1px solid $line-smooth-color;
    cursor: pointer;
    &:hover,
    &:focus {
      border-color: $primary-color;
    }
    &:first-of-type {
      margin-right: 0;
    }
  }
  &__text {
    color: $darker-color;
    font-weight: normal;
  }
  &--disabled {
    .global-actions__button,
    .global-actions__select {
      pointer-events: none;
      opacity: 0.5;
    }
  }
}
.global-actions__text {
  margin: 0;
  span {
    font-weight: bold;
    color: $primary-color;
  }
}
</style>
